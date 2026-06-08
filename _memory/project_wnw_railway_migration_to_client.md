---
name: project-wnw-railway-migration-to-client
description: "WNW Railway переезд с твоего аккаунта на аккаунт клиентки (Womennetworking club, Hobby план) — новые URL, project IDs, deploy CLI"
metadata: 
  node_type: memory
  type: project
  originSessionId: c70a7bfd-290f-4261-a95a-76196adbb7ea
---

WNW wnw-api + wnw-cron переехали с твоего Railway (`gnilosiranton@gmail.com`, `Antthonyyy's Projects`) на аккаунт клиентки `clubwomennetworking@gmail.com` (workspace `Womennetworking club's Projects`, план Hobby $5/мес). Дата: 2026-05-26.

**Why:** клиентка хочет владеть инфрой и платить за неё сама.

**How to apply:**
- При любых правках в `/Users/mac/Downloads/wnw crm/wnw-api/` или `wnw-cron/` — деплоить через `RAILWAY_TOKEN=<token> railway up --service NAME --ci` в её workspace, токены лежат локально (см. ниже). Свой Railway CLI остаётся под твоим аккаунтом — переключать не надо.
- При смене env vars — `railway variables --service NAME --set KEY=VAL` с её токеном. RAILWAY_* не трогать.
- Старые проекты на твоём аккаунте Антон снёс 2026-05-26 (после verify).

## Новая инфра

| Что | Project ID | Service ID | Public URL |
|---|---|---|---|
| wnw-api | `a81373a6-5bd9-496e-9810-8fc6733552ec` | `f39b7e8a-4636-4b6b-af27-aacd56e9b5e6` | `https://wnw-api-production-ae17.up.railway.app` |
| wnw-cron | `a8c2cece-f6a9-440d-9a33-cdb896c31a33` | `36b439e2-63c5-4e83-846f-175f0d484066` | (cron `0 */2 * * *`) |
| tg-outreach-listener | (внутри wnw-api project) | `1405d447-1219-4a66-8aa1-a018cd2aa9dd` | (Telethon listener, 24/7) |
| tg-outreach-cron | (внутри wnw-api project) | `10f22df5-7833-4272-a438-4c1f347fbf5d` | (APScheduler: send 11:00 Kyiv + newcomers sync 8-23 каждые 30 мин) |

Project token клиентского `wnw-api` лежит в `.env::RAILWAY_TOKEN_WNW_CLIENT` (git-crypt'нут, персистентен между перезагрузками). Создан 2026-05-31 через UI Railway (`Settings → Tokens` проекта `a81373a6`), имя `claude-outreach-check-2026-05-31`. Использовать: `RAILWAY_TOKEN=$RAILWAY_TOKEN_WNW_CLIENT railway <cmd>`. Старый путь `/tmp/railway-migration/tokens.env` deprecated — стирался при перезагрузке.

## ⚠️ КРИТИЧНО (обновлено 2026-05-28): два аккаунта Railway, токен эфемерный

- **Railway — ДВА аккаунта.** Прод WNW (listener/cron) — на КЛИЕНТСКОМ `clubwomennetworking@gmail.com`, проект wnw-api = **`a81373a6`**, listener service = `1405d447`. Деплой ТОЛЬКО клиентским токеном: `RAILWAY_TOKEN=<client> railway up --service NAME --ci`.
- **CLI по умолчанию залогинен в ЛИЧНЫЙ `gnilosiranton`** — `railway list` показывает только `Antthonyyy's Projects`, клиентский воркспейс оттуда НЕ виден. Линк по умолчанию ведёт на СТАРЫЙ личный `wnw-api = cd7c8bfe` — это НЕ прод клиентки.
- **Токен теперь в `.env::RAILWAY_TOKEN_WNW_CLIENT`** (git-crypt'нут, персистентен). Старый путь `/tmp/railway-migration/tokens.env` — deprecated, стирался при перезагрузке. Подгрузка: `set -a; source .env; set +a` → `RAILWAY_TOKEN=$RAILWAY_TOKEN_WNW_CLIENT railway <cmd>`.
- **Расхождение, требует проверки:** хотя выше сказано «старые личные проекты снесены 2026-05-26», на 2026-05-28 личный `cd7c8bfe` ЖИВ и слал прод-рассылку (15 шт, дедуп ОК). Похоже, личный listener так и не выключен и фактически работает параллельно/вместо клиентского. Перед любыми действиями — клиентским токеном проверить `a81373a6`: бежит ли там listener, что с volume, кто реально шлёт. Риск AuthKeyDuplicated если оба listener'а на одной TG-сессии.
- **ПЕРЕД любым Railway-действием:** `railway whoami` + `railway list` + `railway status` — убедиться, какой аккаунт/проект под рукой. Личный vs клиентский легко перепутать (один и тот же CLI).

## Что переключилось

- **SmartSender** (проект `zapusk-kluba-300953`) — обновлены webhook URL в 2 воронках:
  - `1331893` «запись на нетворкинг» → `/smartsender/register-networking`
  - `1361726` «запись на бизнес-клуб» → `/smartsender/register-business-club`
- **Vercel** (`wnw-crm-app`):
  - env `WNW_API_URL` → `https://wnw-api-production-ae17.up.railway.app`
  - `next.config.mjs` CSP `connect-src` обновлён на новый URL (захардкожен)

## Что НЕ перенесено (намеренно)

- `SMARTSENDER_FUNNEL_DOGON`, `MANAGER_TELEGRAM_*` — их и на старом не было. См. [[project_wnw_cron_missing_env]].
- Воронка `1360217` из старой карты — в SmartSender выдаёт «Произошла ошибка», по логам wnw-api не дёргалась. Видимо deprecated. См. [[reference_wnw_smartsender_funnels]] (запись устарела).
