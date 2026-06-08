---
name: reference-womennetworking-qr-login
description: "Логин @WomenNetworking (outreach, +380979946252) — SMS/app-код НЕ приходит, заходить только через QR-login"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6701fab6-0894-44cd-a5c2-8d6ac332f6b4
---

Аккаунт **@WomenNetworking** (он же @Tatiana_Zakharov, id `8442538511`, номер `+380979946252`) — серверный outreach-аккаунт, живёт на Railway через `TG_STRING_SESSION`. При попытке свежего логина **SMS/app-код не доставляется** (нет официальной клиентской сессии, где код отрендерится; SMS/flash-call упираются в `SendCodeUnavailableError`). Это **повторяющаяся** проблема — было 2026-05-27 и 2026-06-08.

**Рабочий путь — QR-login** (Telethon `qr_login()`), НЕ код:
1. Свежая отдельная сессия (новый auth-key — Railway-листенер K1 не падает; файл/тот же string = AuthKeyDuplicated, нельзя).
2. `qr_login()` → рендер QR в PNG (`qrcode` lib) → `open` в Preview. Антон сканирует с устройства, где аккаунт залогинен: моба **Настройки → Устройства → Привязать устройство / Scan QR Code** (Desktop сканировать НЕ умеет).
3. QR живёт ~25-55с → авто-`recreate()` в цикле.
4. После скана Telegram просит **2FA cloud password** → его даёт Антон (тот же, что использовали 27.05). Скрипт подставляет `sign_in(password=...)` сам.
5. Готовый скрипт-шаблон лежал в `/tmp/orbita_qr_login.py` (пароль из env `WNW_2FA`, не в файле).

**Перед запуском:** `pgrep -f telethon|tg_outreach|qr_login` — не должно быть живых процессов с тем же auth-key (AuthKeyDuplicated-гард).

**Не делать:** не использовать `tg_outreach.session`/`TG_STRING_SESSION` локально (общий key с Railway → выбьет прод-листенер); не переключаться на личный @AntonGgn без OK ([[feedback_no_switch_to_personal_account]]). Парсинг групп этим аккаунтом = read-only `tg_outreach.py export --session <qr-session> --group <id>`.
