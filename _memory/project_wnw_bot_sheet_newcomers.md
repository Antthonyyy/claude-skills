---
name: project_wnw_bot_sheet_newcomers
description: "WNW лист «Новички» — дебютанты нетворкинга из SmartSender-вебхука, разделение first/repeat по воронке"
metadata: 
  node_type: memory
  type: project
  originSessionId: b784043b-6f62-46ee-9c27-6b904b602478
---

Таблица «бот» Татьяны (Google Sheets, ID `1yzfpvoGZIUQAsxaBVNI5D58Whr8tjGQjSN7TndDqNmY`, владелец `club.best.podrug@gmail.com`) — наш сервис-аккаунт `targetolog-sheets@chat-bot-carma.iam.gserviceaccount.com` имеет роль **writer** (НЕ read-only). Антон (`gnilosiranton@gmail.com`) добавлен редактором 2026-05-23.

**Лист «Новички — ближайший нетворкинг»** (gid 34382443; имя вкладки НЕ переименовывали): дебютанты ближайшего события. Скрипт `scripts/sync_networking_newcomers.py`, cron `*/30 8-23`, лог `/tmp/sync_nw_newcomers.log`. Пишет ТОЛЬКО свой лист, чужие не трогает. Самоочистка — «ближайшее событие» (public.events) катится вперёд.

**С 2026-06-02 лист двухсобытийный** (deployed, проверено по листу + логам cron). Показывает дебютантов ДВУХ ближайших бесплатных входов одним списком + колонка «Событие» (тип+дата): нетворкинг (среда) и бизнес-завтрак (суббота). Заголовок теперь «Новички ближайших бесплатных событий», 9 колонок. Нетворкинг-самоочистка — после среды, завтрак — после субботы (события катятся независимо). Реализация: `EVENTS` + `NETWORKING_NEWCOMERS_SQL` / `BREAKFAST_NEWCOMERS_SQL` (общий `_ENRICH_TAIL`).
- **Новичок завтрака** определяется ПО ИСТОРИИ (у завтрака нет source — вебхук `register-business-club` пишет только `event_id`): нет ни одной регистрации/посещения ЛЮБОГО формата (`registered_networking/networking_attended/breakfast_registered/breakfast_attended`) до момента breakfast-регистрации. Ключ — `details LIKE 'event_id=<ближайший breakfast>'` (надёжен с 04.05: 100% завтраков несут event_id; 2 старые записи без него — ручные backfill `sync_smartsender`/`enrich_from_bot_sheet`).
- **Осознанная асимметрия v1:** нетворкинг-ветка осталась на source-логике и НЕ смотрит историю завтрака. Кросс-форматная защита «первое бесплатно = одно суммарно» применяется только к завтрак-ветке.
- **Риск/техдолг:** дормантный `services/ss_webhook/main.py` (этот репо, не прод) пишет breakfast БЕЗ event_id — при подмене прод-пути (wnw-api) ключ протечёт → завтраки выпадут (safe omission, не ложные новички).
- Деплой: `bash scripts/deploy_outreach_railway.sh cron` клиентским токеном (`RAILWAY_TOKEN_WNW_CLIENT`), сервис `tg-outreach-cron`. Правка не закоммичена (ветка `feat/wnw-saturday-breakfast`), прод взял из рабочего дерева.

**Источник = БД (public.contact_activity), наполняется вебхуком SmartSender** `register-networking` (wnw-api). НЕ Google-листы (лист «нетворкинги» — ручной, заброшен после 06.05; лист «регистрации на нетворкинг» — Telegram-бот, киевское время, неполный).

**Разделение first/repeat по воронке** (идея Антона, надёжнее истории — у БД пробел до 27.04):
- полная воронка SmartSender 1333159 «запись на нетворкинг (каждую среду)» = ПЕРВАЯ регистрация, шлёт `source=smartsender_networking_wednesday`;
- сокращённая воронка (2-я и далее) = повтор, должна слать `source=smartsender_networking_repeat`.
- Вебхук: POST `https://wnw-api-production.up.railway.app/smartsender/register-networking`, header `X-SmartSender-Secret`.
- Правка wnw-api 2026-05-23: `log_activity` пишет `details=f"event_id=...;source={req.source}"` (раньше source терялся). Задеплоено через `railway up` (project/service `wnw-api`).
- Скрипт: новичок = регистрация на ближайшее событие из полной воронки (или до-деплойная без source + нет истории).

**Открытый момент:** сокращённую воронку нужно настроить на `source=smartsender_networking_repeat`, иначе повторы из полной воронки попадут в новички. [[feedback_railway_manual_deploy_per_service]] [[project_wnw_crm_migration]]
