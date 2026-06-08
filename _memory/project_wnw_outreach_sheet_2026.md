---
name: project_wnw_outreach_sheet_2026
description: "WNW Outreach — Рассылка 2026 (sheet for менеджер Татьяны): cron-заливка + автоскрытие прошедших event-листов"
metadata: 
  node_type: memory
  type: project
  originSessionId: 0bb3748e-998f-40bd-991e-f6af13f651c9
---

Google-таблица «WNW Outreach — Рассылка 2026» (ID `1XJhBWQkRrlRvix1PprlmiSGvKFSAjq_QCNEq3rLAheM`, env `WNW_OUTREACH_SHEET_ID`) — рабочая таблица менеджера Татьяны на период пока CRM не доделана. Сервис-аккаунт `targetolog-sheets@chat-bot-carma...` — writer.

**Why:** менеджер должна видеть таблицу без шума — прошедшие нетворкинги и мои технические views (`v_*`) её отвлекают. CRM в работе, до её сдачи это основной операционный интерфейс.

**How to apply:**

С 2026-05-27 крон переехал с Mac на Railway, сервис `tg-outreach-cron` (project `wnw-api`/Womennetworking club, id `cd7c8bfe-...`). APScheduler внутри `scripts/outreach_scheduler.py`, расписание Kyiv-time:

| job | trigger | что делает |
|---|---|---|
| `sync_newcomers` | `8-23 */30` | лист «Новички» в [[project_wnw_bot_sheet_newcomers]] (другая таблица «бот») |
| `sync_networking_regs` | `8-23 15,45` | листы «Нетворкинг — Регистрации» (8 колонок: #, Имя, Когда, Тип, Источник, Телефон, Город/Страна, @ник) + «Участники по странам» |
| `reconcile_public` | `:13` | dual-write outreach→public, дублируется с локальным `cron_reconcile_public.sh` (идемпотентно) |
| `export_views_to_sheets` | `:15` | заливает 5 SQL-views (`v_funnel_overall`, `v_template_performance`, `v_alerts_summary`, `v_daily_activity`, `v_contacts_full`) + `hide_past_event_sheets.py` |

ENV (Railway): `SUPABASE_DB_URL`, `WNW_OUTREACH_SHEET_ID`, `GOOGLE_SA_JSON`, `OUTREACH_BACKEND=postgres`, `DUAL_WRITE_PUBLIC=1`, `RAILWAY_DOCKERFILE_PATH=Dockerfile.outreach-cron`.

**Деплой:** `Dockerfile.outreach-cron` нужно поднимать через минимальный build context (`/tmp/outreach-cron-build/` + свой `railway.toml`), потому что root `railway.toml` форсит `Dockerfile.rawform` и multi-service `[services.NAME.build]` блоки Railway игнорирует при `railway up`. Скрипт см. ниже / `scripts/deploy_outreach_railway.sh`.

**Локальный крон отключён** (Mac, 2026-05-27): убраны три строки из crontab (`cron_sync_networking.sh`, `sync_networking_newcomers.py`, `cron_export_views.sh`). Backup в `/tmp/crontab_backup_*.bak`. LaunchAgent `com.anton.targetolog.active-host.plist` оставлен — пишет hostname в guard-файл при логине (страховка для оставшихся локальных задач: reconcile, follow_ups, nw_reminder, batches_tomorrow, fb_refresh, watchdog_listen).

**Видимые листы для Татьяны:** `Нетворкинг — Регистрации`, `Рассылка`, `Участники по странам`, `Не отправлено`, `WA — Отправить`.

**Если нужно вернуть скрытый лист** — Google Sheets → меню `View → Show hidden sheets`. Данные не удаляются, только hidden.

**Команды:**
- `python scripts/hide_past_event_sheets.py --dry-run` — план без изменений
- `python scripts/hide_past_event_sheets.py --hide-internal-views` — повторно спрятать v_* если кто-то их вернул

Связано: [[reference_outreach_sqlite]] (SQLite-источник для views), [[project_wnw_bot_sheet_newcomers]] (другая таблица «бот» Татьяны, не путать).
