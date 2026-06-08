---
name: WNW Cron — незаданные env vars (напомнить потом)
description: Две cron-джобы на Railway wnw-cron пропускаются/молчат из-за отсутствия env переменных — Антон отложил на потом 2026-04-21
type: project
originSessionId: a81ee1d8-7a87-4a15-95ab-141b667e2736
---
На Railway `wnw-cron` (проект `e9e073a5-7fc7-4d00-bd1e-144f27e0de2c`, сервис `6869f05d-1c5a-4826-82c3-08cd44946eab`) **НЕ заданы**:

1. **`SMARTSENDER_FUNNEL_DOGON`** — `dogon_unpaid` джоба при каждом тике пишет WARN и скипается. Получить funnel ID в SmartSender → Настройки → Воронки.
2. **`MANAGER_TELEGRAM_BOT_TOKEN`** + **`MANAGER_TELEGRAM_CHAT_ID`** — `check_expiring_packages` бежит, но алерты Татьяне об истекающих пакетах резидентов не шлёт (молчаливый skip).

**Why:** 2026-04-21 Антон явно сказал «оставим на потом, напомни мне потом» — не блокер для текущей миграции SmartSender → public schema. Новые джобы `sync_smartsender_contacts` и `sync_smartsender_messages` этих переменных не требуют и работают нормально.

**How to apply:** при следующей работе с wnw-cron или планировании релизов — напомнить Антону добить эти два пункта. Задать через:
```
cd "/Users/mac/Downloads/wnw crm/wnw-cron"
railway variables set SMARTSENDER_FUNNEL_DOGON=<id>
railway variables set MANAGER_TELEGRAM_BOT_TOKEN=<token>
railway variables set MANAGER_TELEGRAM_CHAT_ID=<chat_id>
```
Перед рекомендацией перепроверить через `railway variables` — возможно уже закрыто.
