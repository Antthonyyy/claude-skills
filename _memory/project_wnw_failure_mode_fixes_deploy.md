---
name: project_wnw_failure_mode_fixes_deploy
description: WNW listener — 6 pre-mortem failure-mode фиксов задеплоены на прод 2026-05-31 (commit 1bb8c8b5)
metadata: 
  node_type: memory
  type: project
  originSessionId: d3de0a95-57fb-464d-852b-5761d1fd6219
---

2026-05-31 задеплоены на прод 6 defensive-фиксов WNW-листенера (commit `1bb8c8b5` на ветке `feat/wnw-saturday-breakfast`, запушен в origin). Источник — pre-mortem через `/the-fool`, 3 раунда внешнего ревью + дозакрытие attribution gap.

**Фиксы:** F1 ai_reply_ex со статусом (api_error→manual_hold+алёрт), F2 гидрация атрибуции бот↔менеджер из БД (template-фильтр, см. [[project_wnw_bot_out_template_invariant]]), F3 fail-closed дедуп рассылки, F4 fail-fast при незагруженных модулях промта, F6 +12 предоплатных HOT_TRIGGERS. Тесты: 392 passed.

**Деплой:** Railway клиентский проект wnw-api `a81373a6`, сервис `tg-outreach-listener` id `1405d447-1219-4a66-8aa1-a018cd2aa9dd` (НЕ `...-c5efea9b9b89`, как ошибочно считалось раньше). Деплой через project token (`railway up --service <id> --ci`), autosync OFF. Source repo = None → только `railway up`.

**Подтверждено рантайм-логами:** `identity-guard OK id=8442538511 @Tatiana_Zakharov`, `БД: ВКЛ Postgres (Supabase.outreach)`, `Слушаю входящие`. F2 сработал в проде: входящее от Гульнур → `Менеджер ведёт диалог — авто manual_hold, бот молчит` (тот самый кейс из пре-мортема).

**Грабли деплоя (на будущее):**
- `railway whoami` НЕ работает с project token («Unauthorized») — это by design, токен не привязан к user. Для гейта использовать `railway status` (показывает Project/Environment) + факт, что токен создан в Settings именно клиентского проекта.
- `~/.railway/config.json` был битый («Unable to parse / Login state corrupt»). Убрал секцию `user` (бэкап `config.json.bak-*`), CLI регенерировал. Личный railway-логин был протухший ещё до этого.
- Деплой льёт рабочее дерево → перед `railway up` обязателен `git stash -u` (убрать WIP Антона: payment_intake, group_monitor), после upload — `git stash pop`. Классификатор блокирует деплой грязного дерева — это правильно.
- Project token создаётся в UI: project Settings → Tokens (НЕ account/tokens). Значение показывается один раз. После использования отзывать (Delete в той же таблице).

Связано: [[project_wnw_railway_migration_to_client]], [[project_wnw_outreach_railway_split]], [[feedback_railway_manual_deploy_per_service]].
