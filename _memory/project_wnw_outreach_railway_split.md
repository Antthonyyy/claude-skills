---
name: wnw-outreach-railway-split
description: "WNW outreach на Railway — listener держит ВСЕ TG-операции, cron только non-TG. Иначе AuthKeyDuplicated."
metadata: 
  node_type: memory
  type: project
  originSessionId: addc624e-a553-4273-9187-ea1e0f09ae99
---

# WNW outreach на Womennetworking club (wnw-api проект)

Два сервиса в проекте `wnw-api` (Railway, Womennetworking club workspace):

| Сервис | Что внутри | Dockerfile |
|---|---|---|
| `tg-outreach-listener` | `tg_outreach.py listen` + `outreach_scheduler_tg.py` в фоне (один контейнер, один процесс на Telethon) | `Dockerfile.listener` |
| `tg-outreach-cron` | `outreach_scheduler.py` — только non-TG: newcomers / regs / reconcile / export | `Dockerfile.outreach-cron` |

**Why:** Telegram палит `AuthKeyDuplicatedError` при работе одной сессии с двух разных IP. listener и cron на Railway — разные контейнеры на разных IP. Если оба используют один `TG_STRING_SESSION` и оба коннектятся к Telegram → сессия аннулируется, нужна перегенерация (phone+SMS+2FA).

Все 4 TG-задачи (send_daily 11:00, follow_ups каждый час, nw_reminder среда 12:30, post_networking_check четверг 10:00) запускаются `outreach_scheduler_tg.py` внутри listener-контейнера subprocess'ом. subprocess наследует env (`TG_STRING_SESSION`) и работает с того же IP — никаких конфликтов.

**How to apply:**
- Любая новая TG-задача (новый scripts/send_*.py) добавляется как job в `outreach_scheduler_tg.py`, файл COPY-ится в `Dockerfile.listener`.
- Любая новая non-TG задача — в `outreach_scheduler.py` + COPY в `Dockerfile.outreach-cron`.
- НЕ запускать `listen_wrapper.sh` локально на Mac, пока active на Railway: тут же AuthKeyDuplicated → перевыпуск сессии. Если нужно локально — сначала `railway down --service tg-outreach-listener`.
- Project Token Womennetworking club workspace: создавать в `Settings → Tokens` (формат UUID, не путать с workspace ID).

## Деплой

CLI `railway up --service tg-outreach-X` использует root `railway.toml` `[build]`, **игнорируя** per-service Config Path в Settings UI. Workaround: временно подменить root `railway.toml` на нужный Dockerfile перед каждым `railway up`, потом восстановить из `/tmp/railway.toml.bak` (см. историю в `deploy/README.outreach-railway.md`).

## Env

- `OUTREACH_ENABLE_COLD_SEND=1` — только на listener (там живёт send_daily).
- `OUTREACH_ENABLE_FOLLOW_UPS=1` — только на listener (там живут follow_ups / nw_reminder / post_check).
- `DUAL_WRITE_PUBLIC=1` — на cron (для reconcile_public).
- Сессия: общая `TG_STRING_SESSION` на обоих, регенерится через `python scripts/tg_outreach.py print-string` после удаления `tg_outreach.session*` (требует phone + SMS-код + 2FA).

## Дедуп cold-send живёт ТОЛЬКО в JSON на volume (не в БД)

`cmd_send` (tg_outreach.py ~1152) дедупит исключительно по `LOG_DIR/<contacts_stem>_sent.json`, где `LOG_DIR=OUTREACH_LOG_DIR=/data/outreach_logs` (volume). **По БД дедупа нет.**

**Why:** При сплите на Railway (2026-05-27) volume `/data/outreach_logs` поднялся пустой — миграция НЕ залила `untouched_2026-05-11_sent.json` (он в `.railwayignore`, в образ не попадает). Listener на старте показал `📋 Загружено контактов с рассылки: 0`. Если бы не поймали — авто-send 11:00 28.05 ушёл бы по верхушке CSV заново → дубли ~172 уже отконтаченным + риск PeerFlood. Залили локальный файл (176 ключей, sha256 6c265a6c…) через `railway ssh ... "echo '<base64>' | base64 -d > /data/..."` (stdin railway ssh НЕ пробрасывает — только аргумент-команда; base64 в одну строку < 128KB влезает в один arg).

**How to apply:**
- После ЛЮБОГО пересоздания volume / нового контейнера / нового файла контактов — проверить `railway ssh <listener> "ls /data/outreach_logs/*_sent.json"` и при отсутствии залить sent-лог ДО первого 11:00, иначе дубли. Файл читается в момент запуска job'а, не при старте контейнера — перезапуск не нужен, достаточно положить файл.
- Долгосрочный fix (не сделан): дедуп в `cmd_send` по `outreach.messages` (БД переживает потерю volume). Фрагильность volume-only — техдолг.
- listener-сторона (reply-matching) от пустого volume не страдает: `all_sent` дозаполняется из БД каждые 5 мин (tg_outreach.py ~2539).

Связано: [[wnw-outreach-rules]], [[verification-discipline]], [[Проверять дубли перед повторным запуском рассылки]], [[project_wnw_outreach_daily_send]].
