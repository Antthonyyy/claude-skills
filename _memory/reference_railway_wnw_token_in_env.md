---
name: reference-railway-wnw-token-in-env
description: Railway client project-token wnw-api уже лежит в зашифрованном .env как RAILWAY_TOKEN_WNW_CLIENT — не спрашивать каждый раз
metadata: 
  node_type: memory
  type: reference
  originSessionId: 977aea9c-a6a3-4441-a533-074f9eb62349
---

В корневом `.env` репо `targetolog-ai-assistant` (git-crypt зашифрован) лежит:
- `RAILWAY_TOKEN_WNW_CLIENT` — project-token клиентского workspace «Womennetworking club», project `wnw-api` (`a81373a6-5bd9-496e-9810-8fc6733552ec`).

**Использовать без вопросов:**
```bash
set -a && source .env && set +a
export RAILWAY_TOKEN="$RAILWAY_TOKEN_WNW_CLIENT"
railway status --json
railway logs --service tg-outreach-listener
railway logs --service tg-outreach-cron
railway variables --service tg-outreach-listener
railway redeploy --service <name>   # для деплоя
```

Этого токена хватает для:
- `railway status/logs/variables/redeploy` по любому сервису проекта wnw-api
- `railway up --service <name>` (доказанный путь для деплоя — см. [[project_wnw_failure_mode_fixes_deploy]])

Этого токена НЕ хватает для:
- GraphQL запросов через `Authorization: Bearer $TOKEN` к backboard (возвращает `Not Authorized`) — для этого нужен **account-token** (создаётся в браузере, см. [[reference_railway_client_account_token]]; пишется в `/tmp/rw_account_tok`, стирается при перезагрузке).
- Доступа к другим проектам клиента (wnw-cron `a8c2cece` и трём прочим) — нужен либо отдельный project-token, либо account-token.

**Правило:** прежде чем ныть про токен — `grep RAILWAY .env`. Если есть — использовать.

Связано: [[reference_railway_client_account_token]], [[project_wnw_railway_migration_to_client]], [[feedback_railway_manual_deploy_per_service]].
