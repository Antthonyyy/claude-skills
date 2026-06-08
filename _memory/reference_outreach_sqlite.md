---
name: WNW outreach — SQLite long-term память
description: SQLite БД для контекста бота, аналитики воронки и алертов менеджеру (Phase 1, апрель 2026)
type: reference
originSessionId: 60cb9c92-2f01-456f-8bbc-c1faed0a5cb6
---
С 19 апреля 2026 в Telegram-аутриче WNW работает SQLite-хранилище для long-term памяти бота.

**Файл БД:** `outreach_logs/outreach.db` (WAL-режим). Отдельно от `tg_outreach.session` — никаких lock'ов.

**Модули:**
- [`scripts/outreach_db.py`](../../scripts/outreach_db.py) — DDL + CRUD (`upsert_contact`, `add_message`, `add_event`, `set_stage`, `add_alert`, `build_context_block`).
- [`scripts/migrate_outreach_to_db.py`](../../scripts/migrate_outreach_to_db.py) — одноразовая миграция из `outreach_logs/*_sent.json`.

**5 таблиц:** `contacts`, `messages` (UNIQUE на tg_msg_id), `events` (networking_attended/bought_package/etc), `funnel_stages` (1:1 текущая стадия), `manager_alerts` (meeting_request/hot_lead/partnership/price/negative).

**Интеграция в [`scripts/tg_outreach.py`](../../scripts/tg_outreach.py):**
- listen и send автоматически пишут все сообщения, стадии и алерты в БД.
- `ai_reply(text, history, contact_id=user_id)` подкладывает «КОНТЕКСТ ПО КОНТАКТУ» из БД в системный промт — бот помнит стадию воронки, события и заметки менеджера.

**CLI:** `python scripts/outreach_db.py init` (создать схему), `python scripts/outreach_db.py stats` (счётчики).

**Полная документация и changelog:** [`projects/wnw/WNW_Outreach_Ops.md`](../../projects/wnw/WNW_Outreach_Ops.md), раздел 10.

**Когда использовать память:**
- При вопросах «как помнит ли бот контекст переписки?» / «куда пишутся события клиента?» / «как добавить событие "была на встрече"?»
- При расширении функционала: события, follow-up автоматизация, аналитика воронки.

**Phase 2 (19.04) сделано:** CLI команды (`show`, `mark-attended`, `add-event`, `set-stage`, `note`, `alerts`, `handle-alert`), cron `scripts/run_follow_ups.py` (auto-message через 24-72ч после `networking_attended`, шаблоны без AI, окно 09-22 Amsterdam).

**Phase 3 (19.04) сделано:** SQL-views (`v_funnel_overall`, `v_template_performance`, `v_alerts_summary`, `v_daily_activity`, `v_contacts_full`), HTML-дашборд `scripts/generate_dashboard.py` → `outreach_logs/dashboard.html` (KPI + воронка + A/B + активность + pending алерты + события).

**Фиксы качества промта (19.04):**
- В `_TATIANA_SYSTEM` добавлена секция «ИНТЕРПРЕТАЦИЯ КОНТЕКСТА» — правила по каждой стадии (после `sent_first_touch` НЕ представляться снова; после `attended` спросить про впечатления; запрет квалифицирующих вопросов выше `not_replied`).
- `scripts/backfill_messages_from_telegram.py` одноразово залил 297 реальных сообщений из Telegram, удалил 93 заглушки миграции.

**Установка cron (рекомендуется):**
```cron
5 7-20 * * * cd /Users/mac/targetolog-ai-assistant && venv/bin/python scripts/run_follow_ups.py >> /tmp/follow_ups.log 2>&1
*/15 * * * * cd /Users/mac/targetolog-ai-assistant && venv/bin/python scripts/generate_dashboard.py >> /tmp/dashboard.log 2>&1
```

**Phase 4 (не сделано, опционально):** Streamlit live-dashboard, экспорт views в Sheets, API через @Gnilosirbot для пометки событий, атрибуция шаблонов задним числом.
