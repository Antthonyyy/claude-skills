---
name: project_wnw_bot_out_template_invariant
description: WNW бот — любой bot-out в messages ОБЯЗАН нести template (иначе ломается F2-атрибуция бот↔менеджер)
metadata: 
  node_type: memory
  type: project
  originSessionId: d3de0a95-57fb-464d-852b-5761d1fd6219
---

В WNW-outreach атрибуция бот↔менеджер (`outreach_db.get_out_tg_msg_ids`, фикс F2 от 2026-05-30) считает bot-origin как `direction='out' AND template IS NOT NULL`. Причина: `scripts/backfill_messages_from_telegram.py` тянет историю Telegram и пишет ВСЕ исходящие (включая ручные ответы менеджера из Telegram-app) как `direction='out'` с `template=NULL`. Поэтому `template IS NOT NULL` = «уверенно бот», `NULL` = backfill/менеджер/неуверенно → исключаем (fail-safe: бот промолчит, а не влезет в ручной диалог).

**Инвариант:** КАЖДЫЙ новый bot-send путь, пишущий в `messages`, ОБЯЗАН ставить `template=...` в `add_message(direction="out", ...)`. Сейчас помечены все: листенер (`bot_reply`), cold-send/reminder/offline/berlin/post_nw (имя варианта/события), night-queue (`ai_reply_night_queue`).

**Почему легко проколоться:** тесты проверяют поведение `get_out_tg_msg_ids`, а НЕ все call sites. Новый sender-скрипт без `template=` пройдёт CI, но даст ложный `manual_hold` (последний out сочтётся менеджерским). Проверять AST-аудитом: все `add_message(direction="out")` имеют `template`, кроме backfill.

**Анти-паттерн (НЕ делать):** инвертировать на «всё out = бот, кроме backfill-метки» — забэкфилленные ручные ответы менеджера (template=NULL, уже на проде) прошли бы как бот → бот влезает в ручной диалог. Хуже исходного бага + требует миграции истории.

Связано: [[project_wnw_bot_prompt_architecture]], [[project_wnw_manager_grace]].
