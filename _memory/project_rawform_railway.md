---
name: Rawform — Railway деплой
description: rawform-scheduler на Railway — автосинк Meta Ads → Google Sheets каждый час, weekend pause/resume, Hobby план
type: project
---

## Railway деплой RawForm

**Проект Railway:** welcoming-purpose (ID: 0c979e52-987c-4f4d-b7d9-b942e04a0ce5)
**Сервис:** rawform-scheduler (ID: 6dc202c8-eb22-40f4-a41b-8720091f2a2e) — Online
**План:** Hobby ($5/мес)
**Регион:** europe-west4

### Что делает
- APScheduler синхронизирует Meta Ads → Google Sheets каждый час в :15
- `scripts/rawform_scheduler.py` → `scripts/sheets_reporter.py`

### Env переменные на Railway
- `META_ACCESS_TOKEN`, `META_APP_ID`, `META_APP_SECRET`
- `GOOGLE_SHEETS_CREDENTIALS_JSON` — полный JSON сервисного аккаунта (записывается в /tmp/google_creds.json при старте)

### Google Sheets
- ID: `1BLK8nv-uenhnnh43YcwwsvqiCmv1S1oON1-8fsMBoU8`
- Owner: gnilosiranton@gmail.com
- Сервисный аккаунт: targetolog-sheets@chat-bot-carma.iam.gserviceaccount.com (editor)

### Meta Ads Automated Rules
- Weekend Pause: пт 15:00 Bali (00:00 Pacific)
- Monday Resume: пн 07:00 Bali (16:00 Sunday Pacific)

### Docker
- `Dockerfile.rawform` → python:3.11-slim
- `requirements.rawform.txt` — минимальные зависимости

**Why:** Клиент (Onur) хочет видеть актуальные данные кампаний в Google Sheets без ручного обновления.
**How to apply:** При изменениях в sheets_reporter.py или scheduler — помнить про Railway деплой. Пушить в main автоматически триггерит редеплой.

Дата создания: 2026-03-17
