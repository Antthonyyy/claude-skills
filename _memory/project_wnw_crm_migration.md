---
name: WNW CRM migration — outreach → public через dual-write
description: 21.04.2026 мигрировали 161 активных outreach-контактов в public.contacts (CRM Татьяны). Dual-write работает. Зоны ответственности чатов разделены.
type: project
originSessionId: 60cb9c92-2f01-456f-8bbc-c1faed0a5cb6
---
# WNW: миграция outreach → public CRM (21.04.2026)

## Что произошло
Supabase-проект `uqclcbjfetjdplldevwv` содержит 2 схемы:
- `outreach.*` — наш pipeline (tg_outreach.py, 743 контакта)
- `public.*` — CRM Татьяны (wnw-api, wnw-cron, frontend на Vercel)

До 21.04 жили параллельно без связи. 21.04 построили мост через
`public.contact_identities` (channel='telegram_outreach', external_id=tg user_id).

## Текущее состояние public.*
- contacts: **161** (только с реальным взаимодействием)
- contact_identities: 161
- messages: 571 (channel='telegram_outreach')
- contact_activity: 32
- funnel_stages: 159

## Dual-write
**Why:** `scripts/outreach_core/public_sync.py` зеркалит outreach → public в реальном времени через SAVEPOINT (SQL-сбой mirror не рушит outreach).
**How to apply:** `DUAL_WRITE_PUBLIC=1` в .env; функции: mirror_message / mirror_activity / mirror_stage / mirror_contact_upsert.

Reconcile cron `13 * * * *` досылает drift outreach→public.

## Pipeline vs CRM
**Why:** 581 контакт попал в outreach через enrich_contacts_bio (собрали bio без отправки). Помещать их в CRM-dashboard Татьяны = замусоривать интерфейс пустыми карточками.
**How to apply:** `public.contacts` = **только те, с кем реально писали** (161). `mirror_contact_upsert` создаёт public-запись только при реальном сообщении (create_if_missing=False для bio-enrich путей). Cleanup сделан 21.04 через `scripts/cleanup_empty_public_contacts.py`.

## Разделение чатов
**Why:** 21.04 (поздно вечером) пользователь решил что CRM-разработка идёт в отдельном чате Claude Code в `/Users/mac/Downloads/wnw crm/`, а здесь — только рассылка.
**How to apply:** В этом репо (`/Users/mac/targetolog-ai-assistant/`) НЕ трогать:
- `/Users/mac/Downloads/wnw crm/**` (frontend Next.js + wnw-api + wnw-cron)
- Vercel deploys
- public.* SCHEMA (но писать в неё через dual-write — можно, это данные)

## Security (RLS)
**Why:** Supabase Advisor флагнул CRITICAL на public.* (RLS disabled).
**How to apply:** 21.04 включили RLS на всех public.* и outreach.*. postgres/service_role обходят (pooler, wnw-api, frontend работают). anon deny — проверено grep что anon-key нигде не используется.

## Расписание cron (после 21.04)
- `45 8 * * *` — batches_tomorrow.sh (старт 07:45 Amsterdam, партия 1 в 09:00 Amsterdam)
- `30 12 22 4 *` — send_nw_reminder (разовый на 22.04, 11:30 Amsterdam)
- `13 * * * *` — reconcile_public каждый час
- `5 10-23 * * *` — run_follow_ups
- `15 * * * *` — export_views_to_sheets

## Рассылка — режим
- Партий 5 × 7 = 35/день
- Начало 09:00 Amsterdam, последняя 15:00 Amsterdam
- Ночной стоп 23:00-09:00 Amsterdam для listen
- PeerFlood-лок `outreach_logs/peer_flood_lock.json` с ключом `locked_until` (используется batches_tomorrow.sh + send_nw_reminder.py + run_follow_ups.py)

## Таблица "бот" Google Sheets (менеджерская)
**ID:** `1yzfpvoGZIUQAsxaBVNI5D58Whr8tjGQjSN7TndDqNmY`
**Service account:** `targetolog-sheets@chat-bot-carma.iam.gserviceaccount.com`
**READ-ONLY для нас** — это Татьянина таблица, не править.
Скрипт `scripts/enrich_from_bot_sheet.py` один раз прогнали 21.04 — обогатили 12 совпадений (phone/email/activity/stage). Запускать вручную после обновления таблицы.

## Модель бота
Haiku 4.5 (`claude-haiku-4-5-20251001`) после перевода с Sonnet 21.04.

## WhatsApp-группы
Из промта убрана логика "даём группу до оплаты" — теперь только после оплаты ланча/пакета (21.04, вариант A; Антон уточнит с Татьяной).

## Тесты
82 pytest: set_stage priority (declined=9), got_link dedup (key='type'),
peer_flood lock (key='locked_until'), bio classification, pending_queue atomicity,
offline-payment detector, public_sync SAVEPOINT.

## Snapshot / rollback
`backups/snapshot_20260421-2108/` — CSV на момент миграции.
Git: `snapshot-before-migration-20260421-2028`.
