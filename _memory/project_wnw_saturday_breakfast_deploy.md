---
name: project_wnw_saturday_breakfast_deploy
description: WNW бот — субботний бизнес-завтрак как второй бесплатный онлайн-вход + маршрутизация по дню недели (deployed 2026-05-30)
metadata: 
  node_type: memory
  type: project
  originSessionId: 006302b6-7642-43fc-bfd0-659364f966bd
---

С 2026-05-30 08:02 UTC+3 в клиентском проде wnw-api listener'а (Railway `a81373a6`, service `tg-outreach-listener` `1405d447`, deployment `27649abd`) — два равноценных бесплатных онлайн-входа: нетворкинг (среда 17:00 Ams) + бизнес-завтрак (суббота 10:00 Ams). Раньше завтрак был закрытым перком пакета («не приглашай сама»).

**Маршрутизация по дню недели:** бот зовёт на БЛИЖАЙШЕЕ событие, точка переключения = час старта. Таблица: пн/вт/ср-до-17 → среда; ср-после-17/чт/пт/сб-до-10 → суббота; сб-после-10/вс → среда. В день события — «сегодня». Реализовано в [`scripts/outreach_core/llm_tatiana.py`](../../../../../targetolog-ai-assistant/scripts/outreach_core/llm_tatiana.py) (`_pick_nearest_event`, `next_event_invite_phrase`, блок «БЛИЖАЙШЕЕ СОБЫТИЕ» в контексте времени с динамическим DST tz-пересчётом для каждого события). Рассылка: cmd_send подставляет `{event_when}` в шаблоны v7-v10 ([clients/wnw/tasks/messages/](../../../../../targetolog-ai-assistant/clients/wnw/tasks/messages/)).

**«Первое бесплатно» = одно ознакомительное посещение суммарно** на оба формата (любой). Стадия `attended` = была на любом из двух → на бесплатное больше не зовём.

**Название НЕ меняли** (намеренно): субботнее событие осталось «бизнес-завтрак», чтобы не создавать омоним с платными офлайн-ланчами по городам (25/35/50€). Решение принято в pre-mortem (/the-fool) — переименование создавало бы коллизию офлайн-триггера (упоминание города → платный сценарий) с бесплатным онлайн-тредом.

**Спикер-guard сохранён:** для воронки спикеров на завтрак инициативно НЕ ведём (сначала обычный нетворкинг).

**Why:** клиентка попросила приравнять субботний завтрак к нетворкингу и расширить окно конверсии (пн-сб не один день в неделю, а два).

**How to apply:**
- Промт-модули, где зашита логика: [`knowledge/bot/tatiana_core.md`](../../../../../targetolog-ai-assistant/knowledge/bot/tatiana_core.md) (блок ЦЕЛЬ, ВЫБОР СОБЫТИЯ, факты о событиях, ссылки), [`tatiana_cold.md`](../../../../../targetolog-ai-assistant/knowledge/bot/tatiana_cold.md) (первое касание + возражения), [`tatiana_dialog.md`](../../../../../targetolog-ai-assistant/knowledge/bot/tatiana_dialog.md) (отправка ссылки разведена по событию), [`tatiana_warm.md`](../../../../../targetolog-ai-assistant/knowledge/bot/tatiana_warm.md).
- Перед правкой поведения бота — карта файлов в [[project_wnw_bot_prompt_architecture]].
- Регресс-тест маршрутизации: [`tests/test_event_routing.py`](../../../../../targetolog-ai-assistant/tests/test_event_routing.py) (12 кейсов день×час).
- **Деплой listener'а** (подтверждено фактом 2026-05-30): механизм `railway up` из рабочего дерева (НЕ git-based и НЕ redeploy/restart — последние не подхватят новый код). Гейтить клиентский аккаунт через `railway status` (project: `wnw-api`, id `a81373a6`). Связано с [[project_wnw_outreach_railway_split]] и [[project_wnw_railway_migration_to_client]].
- Ветка: `feat/wnw-saturday-breakfast`, коммит `5a850d4f`. Деплой делался с временной ветки от прод-baseline (`2d163f3f`) cherry-pick'ом, чтобы не утащить недозревшие коммиты `f149f916` (бизнес-ланч reminder, ждёт доработки оффера) и `cf1796be` (helper-refactor follow-up'а).
