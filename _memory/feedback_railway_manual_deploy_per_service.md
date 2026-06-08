---
name: Railway — ручной деплой только затронутого сервиса
description: AutoSync с GitHub в Railway выключен. После push в main — деплоить ТОЛЬКО тот сервис, к которому относятся изменённые файлы. Другие сервисы не трогать.
type: feedback
originSessionId: 09441818-7f2e-419e-94d8-9257bd7a6a24
---
# Railway — ручной деплой только затронутого сервиса

**AutoSync с GitHub в Railway выключен (проверено 2026-04-24).** Push в `origin/main` НЕ триггерит автоматический rebuild/redeploy сервисов Railway. Каждый деплой нужно инициировать вручную.

Проверка статуса autosync для любого сервиса — через GraphQL API (пустой `repoTriggers.edges = []` означает autosync off):

```bash
TOKEN=$(cat ~/.railway/config.json | python -c "import sys,json; print(json.load(sys.stdin)['user']['token'])")
curl -s 'https://backboard.railway.com/graphql/v2' \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"query":"query { project(id: \"<PROJECT_ID>\") { services { edges { node { id name repoTriggers { edges { node { id branch repository } } } } } } } }"}'
```

Состояние на 2026-04-24 для проекта `welcoming-purpose` (ID `0c979e52-987c-4f4d-b7d9-b942e04a0ce5`):

- `rawform-scheduler` → `repoTriggers: []` (autosync off, деплой только через `railway redeploy`)

**Why:** Один репо `targetolog-ai-assistant` обслуживает несколько Railway-сервисов (`rawform-scheduler`, в будущем — outreach listener и др.). При autosync любой push пересобирал ВСЕ сервисы подряд — это:

- лишние ребилды там, где код не менялся (трата минут CI, иногда сбои);
- риск что изменение предназначенное для сервиса A уронит сервис B (если случайно задели общий модуль);
- Антону сложно понимать, какой сервис сейчас в каком состоянии.

**How to apply:**

1. **Перед деплоем определи, какому сервису относятся изменения.** Посмотри `git diff --stat HEAD^..HEAD` — какие файлы поменялись. Сопоставь:
   - `scripts/rawform_scheduler.py`, `Dockerfile.rawform`, `railway.toml` → сервис `rawform-scheduler` (проект `welcoming-purpose`)
   - `scripts/tg_outreach.py`, `scripts/outreach_core/*`, `knowledge/bot/*`, `Dockerfile.listener`, `railway.listener.toml` → outreach listener (сейчас на Mac, НЕ в Railway; если появится Railway-сервис — привязать соответствие)
   - Общие файлы (`scripts/outreach_db.py`, общие утилиты) — подумать какие сервисы их реально импортируют и задеплоить только их.

2. **Если код НЕ касается ни одного Railway-сервиса** (например, только тесты или локальный скрипт) — **ничего не деплоить.**

3. **Если код касается одного сервиса** — деплоить ТОЛЬКО его:

   ```bash
   railway link --project <project> --service <service-name>
   railway redeploy --yes        # запустит пересборку текущего кода из GitHub main
   ```

   `redeploy` использует последний коммит в привязанной ветке репо. Альтернатива — `railway up` (загружает локальные файлы напрямую, без GitHub), но предпочтительнее `redeploy` — синхронизация с git остаётся достоверной.

4. **Если код касается нескольких сервисов** — деплоить по очереди, каждый отдельно, смотреть статус после каждого.

5. **После деплоя ВСЕГДА проверять:**

   ```bash
   railway status --json | python -c "import sys,json; d=json.load(sys.stdin); [print(e['node']['serviceName'], '-', e['node'].get('latestDeployment',{}).get('status'), e['node'].get('latestDeployment',{}).get('createdAt')) for e in d['environments']['edges'][0]['node']['serviceInstances']['edges']]"
   ```

   `status=SUCCESS` + `createdAt` в пределах минут от момента `railway redeploy` = деплой реально прошёл.

**НЕ ТРОГАТЬ** сервисы к которым изменения не относятся. Нет команды типа "задеплой всё подряд" — это было поведение отключённого autosync, и оно намеренно убрано.
