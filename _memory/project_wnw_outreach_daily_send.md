---
name: project_wnw_outreach_daily_send
description: "WNW outreach — авто-рассылка 20/день через LaunchAgent, дата нетворкинга в шаблонах динамическая"
metadata: 
  node_type: memory
  type: project
  originSessionId: ab228724-7e4e-4f41-9d1f-611cb9b9538f
---

Цель Антона: рассылать **~20 контактов/день без простоя** (cold-аутрич WNW). С 23.05.2026 — автоматизировано.

⚠️ ОБНОВЛЕНО 2026-05-29: с 2026-05-27 рассылка НЕ на Mac LaunchAgent, а на Railway — job `send_daily` 11:00 Киев внутри контейнера `tg-outreach-listener` (`outreach_scheduler_tg.py`), workspace Womennetworking club. Прод-БД — Supabase Postgres (`outreach.messages`), не локальный SQLite. Проверка факта рассылки за день: `psql $SUPABASE_DB_URL` → `SELECT ... FROM outreach.messages WHERE direction='out'`. Подробно: [[wnw-outreach-railway-split]]. Текст ниже про время/лимит/дату-плейсхолдер актуален, изменился хостинг.

⚠️ СТАТУС 2026-06-07: список `untouched_2026-05-11.csv` (420) почти сухой — **73 свежих** (330 отконтачены, 46 berlin-исключены, считал прямым запросом к `outreach.messages`/`contacts` со `search_path=outreach`). 5-6 июня была ПАУЗА: прод-env листенера стоял `OUTREACH_ENABLE_COLD_SEND=0` + `OUTREACH_COLD_SEND_LIMIT=4` (не сбой, не PeerFlood). 07.06 вернул ENABLE=1, LIMIT=18 (Антон выбрал мягкий 15-20 вместо 25). **Refill нового списка нужен ~к 11 июня**, иначе рассылка снова уйдёт в 0 по исчерпанию. Env-флаги меняются на КЛИЕНТСКОМ Railway: `RAILWAY_TOKEN=$RAILWAY_TOKEN_WNW_CLIENT railway variables --service tg-outreach-listener --set ...` (смена env = редеплой листенера, новый контейнер читает новый `os.getenv`). Отчёт: `clients/wnw/tasks/orchestrate_outreach_tomorrow_2026-06-07.md`.

**Как было устроено локально (до 27.05, для отката):**
- **Авто-рассылка:** LaunchAgent `com.anton.tg-outreach.send` → `scripts/send_wrapper.sh`, ежедневно **11:00 Киев** (10:00 Ams). `RunAtLoad=false`, не daemon, guard от параллельного запуска (`pgrep tg_outreach.py send`). plist: `~/Library/LaunchAgents/com.anton.tg-outreach.send.plist`.
- **Safety-гейт:** wrapper шлёт ТОЛЬКО при `OUTREACH_ENABLE_COLD_SEND=1` (иначе «🚫 Cold-send отключён» и выход). Флаг прописан в env plist → крон включён. Для ручного запуска: `OUTREACH_ENABLE_COLD_SEND=1 [OUTREACH_COLD_SEND_LIMIT=N] scripts/send_wrapper.sh`.
- **Лимит:** если `OUTREACH_COLD_SEND_LIMIT` не задан — wrapper берёт **случайный 10–15/день** (`RANDOM%6+10`). Разный объём = естественнее, анти-PeerFlood. Снижен с 20 после PeerFlood 16/18.05.
- **Динамическая дата:** в шаблонах `clients/wnw/tasks/messages/v7,v8,v9,v10_bio.txt` плейсхолдер `{next_wed}`. Рендерится в `tg_outreach.py::next_networking_date_ru()` (ближайшая среда, формат «27 мая»), подстановка один раз при загрузке шаблонов в `cmd_send` — ДО per-contact `.format()`. Захардкоженных дат больше нет, протухания нет.
- **Список:** `outreach_logs/untouched_2026-05-11.csv` (420, дедуп через `_sent.json`, приватные сегменты hard-skip). На 23.05 осталось ~245 = ~12 дней. **Когда кончится — cron будет слать 0/день, нужен новый список контактов (refill).**
- Листенер (автоответы) — отдельный LaunchAgent `com.anton.tg-outreach.listener`, 24/7.

**How to apply:**
1. Сменить время рассылки → `Hour`/`Minute` в plist + `launchctl unload && launchctl load -w`.
2. Запускать send вручную всегда с `python -u` — иначе stdout block-буферизуется при `> log`, лог пустой хотя отправка идёт. Факт сверять в `outreach.messages`, не по логу. См. [[feedback_no_duplicate_sends]].
3. Лимит 20 + задержки 30–90с (config) — безопасный темп против PeerFlood.
4. Перед расширением правок в send — `/code-reviewer`. Связано с [[project_wnw_networking_schedule]].
