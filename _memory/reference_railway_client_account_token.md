---
name: reference_railway_client_account_token
description: Railway клиентский account-token claude-cli-2026-05-31 (workspace-scope) — как обращаться к клиентскому Railway без TTY-логина
metadata: 
  node_type: memory
  type: reference
  originSessionId: d3de0a95-57fb-464d-852b-5761d1fd6219
---

В клиентском Railway-аккаунте (Womennetworking club) создан account-token `claude-cli-2026-05-31`, scope = workspace «Womennetworking club's Projects» (видит все 5 проектов: wnw-api `a81373a6`, wnw-cron `a8c2cece`, + три прочих). Создан 2026-05-31 через браузер (railway.com/account/tokens), потому что `railway login` (в т.ч. `--browserless`) требует TTY и из bash-среды без терминала невозможен.

**Где значение:** session-only в `/tmp/rw_account_tok` (chmod 600). `/tmp` стирается при перезагрузке Mac → после ребута токен-файл пропадает, надо пересоздать (новый токен в браузере) ИЛИ Антон сам впишет в `~/.zshrc`/git-crypt `.env` (мне harness блокирует запись клиентского секрета в plaintext-профиль — credential-persistence gate).

**Как пользоваться:**
- GraphQL (надёжнее всего, весь API workspace): `curl -H "Authorization: Bearer $(cat /tmp/rw_account_tok)" https://backboard.railway.com/graphql/v2 -d '{"query":"..."}'`. Запросы `projects`, `service(id)`, `deployments`, `deploymentLogs` работают. Запрос `me` НЕ работает (нужен personal-token — у workspace/project-токенов нет доступа к `me`, это НЕ значит токен невалиден).
- CLI `whoami`/`list` через `RAILWAY_API_TOKEN=...` на CLI 4.31.0 дают Unauthorized (шероховатость CLI), хотя токен рабочий — не биться, использовать GraphQL.
- **Деплой** — по-прежнему project-token через `RAILWAY_TOKEN=<project-token> railway up --service <id>` (этот путь доказанно работает, см. [[project_wnw_failure_mode_fixes_deploy]]).

**CLI по умолчанию НЕ залогинен** ни в личный, ни в клиента (личный config я почистил при деплое 2026-05-31, личная сессия слетела — `railway login` заново под личным, когда понадобится). Это не противоречит [[project_wnw_railway_migration_to_client]]: глобального дефолта на клиента нет, доступ к клиенту — только явным токеном.

Связано: [[project_wnw_railway_migration_to_client]], [[feedback_railway_manual_deploy_per_service]], [[project_wnw_failure_mode_fixes_deploy]].
