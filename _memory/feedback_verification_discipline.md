---
name: feedback-verification-discipline
description: "Дисциплина независимой проверки перед заявлением «работает» — pgrep -f шире, читать каждую строку crontab, делить «проверено» vs «верю что сделал»"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: c70a7bfd-290f-4261-a95a-76196adbb7ea
---

При финальных отчётах о миграциях/деплоях НЕ писать «всё работает» / «верифицировано», пока не выполнено каждое из трёх:

1. **`pgrep -f "<wrapper>"`, не только Python-имя.** Если процесс запускается через bash-wrapper (`listen_wrapper.sh`, `send_wrapper.sh`, `watchdog_*.sh`) — wrapper-процесс может жить отдельно от Python-child'а, особенно если был запущен руками `nohup ... &` помимо LaunchAgent. После `launchctl unload` или `pkill` всегда проверять и Python-имя, и wrapper-имя.

2. **Crontab — читать ЛИНИЯ-ЗА-ЛИНИЕЙ.** Не заявлять «cron пропустит через guard», пока не прочитана каждая строка `crontab -l` и в каждой не подтверждено: либо вызов через wrapper с `source active_host_guard.sh`, либо прямой Python-вызов с явной guard-логикой. Если хоть одна строка вызывает `venv/bin/python script.py` напрямую — guard не сработает, надо оборачивать.

3. **«Проверено» vs «верю что сделал».** Финальный отчёт делить на два раздела:
   - **Verified independently** — что подтверждено через `curl`/`grep`/`ls`/CLI вызов или скриншот, выполненный после действия.
   - **Believed done** — что я сделал, но не перепроверил после.
   Пользователь не должен сам ловить «верю что сделал», выданное как «verified».

**Why:** 2026-05-27 при переезде WNW outreach на Railway я заявил «все 7 проверок зелёные», в отчёт включил «Mac LaunchAgents выключены» и «local cron пропустят». На деле:
- Mac `listen_wrapper.sh` крутился ручным процессом (не через LaunchAgent), `pgrep tg_outreach` его не находил → continued AuthKeyDuplicatedError → я ошибочно пошёл перевыпускать TG-сессию через QR+2FA вместо того чтобы убить wrapper;
- В crontab 6 из 9 cron-задач вызывали Python напрямую без guard → они бы продолжили работать параллельно с Railway, дёргать БД/Sheets, если бы Антон не поймал.

**How to apply:** перед любым финальным отчётом о миграции с локалки на облако — обязательно прогнать чек-лист из 3 пунктов выше. Лучше быть нудным, чем выдать ложное «всё ок».

### Случай 2026-05-27 (Railway tg-outreach-cron, повтор)

Снова словил «верю что сделал» в трёх местах:
1. **LaunchAgent guard** — `launchctl load ... .plist` вернул успех (PID получен), но я не проверил, что `~/.targetolog_active_host` РЕАЛЬНО создан в результате RunAtLoad. Файл был от `active_on.sh` ранее, и я принял его за результат LaunchAgent. После переименования Антоном в `.disabled-*` файл пропал, guarded задачи стали пропускаться. **Правило:** проверять конечный артефакт (файл/строка в БД/HTTP 200), а не возврат intermediate-команды (`launchctl load` exit 0 ≠ агент сделал свою работу).
2. **Railway workspace** — заявил «это Womennetworking club workspace». На деле я залогинен под personal user (`antthonyyy's Projects`), хожу в чужой project по membership через id `cd7c8bfe-...`. Workspace ownership ≠ project membership через user-token.
3. **Состояние crontab** — мой `grep -vE` затронул ровно три строки (подтверждено backup’ом). Но в финале Антон убрал ещё и `watchdog_listen`, `fb_refresh_token`, `batches_tomorrow`, `cron_follow_ups`, `cron_reconcile_public` — моё «вот текущий crontab» в отчёте устарело к моменту чтения. **Правило:** не цитировать «текущий state» как факт, если между моим `crontab -l` и отчётом пользователь мог редактировать вручную; писать «после моей команды crontab был таким».

### Случай 2026-06-04 (git: чистый checkout ≠ грязное рабочее дерево)

Дважды заявил «коммит готов, тесты зелёные», прогоняя pytest в **грязном рабочем дереве** — Антон оба раза поймал, проверяя **чистый архив HEAD**.
1. Частичный `git apply --cached` отрезал хунк с определением CTE `declined_event`, оставив ссылку на него → на чистом checkout `sqlite3.OperationalError: no such table: declined_event`. Локально зелено, потому что в dirty working tree CTE был unstaged.
2. Раньше — коммит содержал тесты на `resolve_followup_types`, но сам `run_follow_ups.py` не был закоммичен → clean checkout падал ImportError.

**Правило:** перед «коммит/PR консистентен» — проверять из **чистого** состояния, не из рабочей директории:
`git archive HEAD | tar -x -C /tmp/check && cd /tmp/check && venv/bin/python -m pytest`.
Число тестов в clean archive МЕНЬШЕ, чем в dirty tree (archive не содержит untracked WIP-файлов) — это норма, не повод тащить untracked в коммит. Связано: [[feedback-git-commit-hygiene]] (никогда `git add -A`; staged ≠ committed ≠ clean-checkout).

**Деплой-следствие (WNW listener):** `deploy_outreach_railway.sh` копирует файлы из CWD → деплоить только из чистого `git archive HEAD`, иначе в прод уедет unstaged WIP (был `payment_intake` в `outreach_db.py`). Сервис `tg-outreach-listener` `1405d447`, проект `wnw-api` `a81373a6`. `timeout` на macOS нет — для ограничения стрима `railway logs` использовать `| head -N` + tool-timeout, не `timeout`.
