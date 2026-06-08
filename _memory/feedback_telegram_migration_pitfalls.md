---
name: telegram-migration-pitfalls
description: "При переносе Telegram-сессии с Mac на Railway три критические грабли — terminate all other sessions ПЕРЕД миграцией, не использовать railway add --variables, force_sms в Telethon deprecated"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 09a585ac-925f-43cc-be06-700258a69d97
---

При миграции Telethon-листенера (outreach-аккаунт) с локального Mac на Railway получили три проблемы. Все три повторяются легко — записываю чтобы не наступить заново.

## 1. AuthKeyDuplicatedError при первом запуске на Railway

**Что произошло (2026-05-27):** На Mac выгрузили LaunchAgent, проверили `ps aux` — нет активных Telethon-процессов. Залили `tg_outreach.session` → StringSession → Railway env → deploy. Контейнер на Railway подключился, Telegram увидел auth_key с другого IP (был активен в моб. Telegram-app outreach-аккаунта **на серверной стороне Telegram**), вернул `AuthKeyDuplicatedError` и **аннулировал ключ**. Пришлось делать re-auth с нуля.

**Why:** Telegram считает auth_key активным пока его не terminate'нули **в Settings → Devices внутри самого Telegram-аккаунта**. Выгрузка локального LaunchAgent не убирает session с server-side. Любая активная Telegram-сессия на аккаунте (Mobile, Desktop, iPad) — отдельный auth_key, и если он "свежий" — добавление нового с другого IP триггерит duplicated.

**How to apply:** Перед заливкой StringSession на Railway (или любой переезд auth_key на другой IP):
1. **Settings → Devices в самом outreach-аккаунте → Terminate All Other Sessions.** Должна остаться только та сессия из которой потом будешь генерить StringSession.
2. Только после этого `print-string` → `--set` → deploy.
3. Если уже получили `AuthKeyDuplicatedError` — ключ revoked, нужен re-auth с нуля.

См. [[telegram-migration-pitfalls]]

## 2. `railway add --service NAME --variables KEY=VAL` echo'ит значения в stdout

**Что произошло:** Команда `railway add --service tg-outreach-listener --variables "TG_STRING_SESSION=..."` интерактивно показала все значения как `> Enter a variable KEY=VAL` в stdout. Все 11 секретов (TG_STRING_SESSION, ANTHROPIC_API_KEY, SUPABASE_DB_URL с паролем) ушли в логи сессии.

**Why:** В Railway CLI 4.x `railway add --variables` запускает интерактивный wizard, который "печатает" вводы.

**How to apply:** Для секретов **никогда не использовать `railway add --variables`**. Правильный паттерн:
1. `railway add --service NAME` — создать пустой сервис.
2. `railway variables --service NAME --skip-deploys --set "KEY=$VAR" >/dev/null 2>&1` — заливать переменные тихо, по одной или через цепочку `--set`.

Флаг `--skip-deploys` важен: не триггерит redeploy при каждом изменении переменной.

## 3. Telethon `send_code_request(phone, force_sms=True)` deprecated и не работает

**Что произошло:** Когда код от Telegram не доходил до Telegram-app, попробовали `force_sms=True` для отправки через SMS. Telethon выдал warning `force_sms has been deprecated and no longer works` и проигнорировал флаг. Telegram сам решил что варианты доставки исчерпаны → `SendCodeUnavailableError`.

**Why:** Telegram больше не позволяет клиенту форсить SMS — он сам определяет канал доставки по числу активных сессий: есть active session → код в app, нет — SMS. После нескольких подряд send_code_request → cooldown 10-30 мин на номер.

**How to apply:** Если кода нет в app — НЕ долбить `send_code_request` повторно (`SendCodeUnavailableError` → cooldown расширяется). Варианты восстановления:
- **QR-login** (`client.qr_login()`) — работает поверх любых cooldown'ов, нужен только активный outreach-Telegram где сканировать. Скрипт-болванка: `/tmp/auth_qr.py` (используется `qrcode` lib).
- Подождать 10-30 мин cooldown'а и сделать обычный `client.start()`.
- Logout outreach из всех клиентов → Telegram вынужден прислать SMS.

См. также [[no-duplicate-sends]].
