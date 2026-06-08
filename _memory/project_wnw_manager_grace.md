---
name: project_wnw_manager_grace
description: WNW autoreply bot defers all replies (manager-grace window) for a second human manager + softer sales ladder
metadata: 
  node_type: memory
  type: project
  originSessionId: 4b5994d9-953f-4fe6-9915-365c3fa981c2
---

С 2026-05-22 в WNW outreach (Telegram Business автоответчик, @Gnilosirbot, Haiku 4.5) параллельно с ботом работает **второй живой менеджер**. Поэтому бот теперь **откладывает все ответы** на «окно приоритета менеджера» и отправляет только если за это окно никто не ответил вручную и нет более нового входящего.

- Обычный диалог: `BOT_REPLY_MANAGER_GRACE_MIN`, дефолт **30 мин**.
- Горячие лиды (`беру`/`как оплатить`, HOT_TRIGGERS) и offline-payment data: `BOT_REPLY_HOT_GRACE_MIN`, дефолт **5 мин** (чтобы не остужать).
- Механизм: `_deferred_reply` (фоновый asyncio-таск, лок НЕ держится во время сна) + `_send_deferred_reply_if_current` (финальная перепроверка `_manager_answered_after`/`_newer_incoming_after`/manual_hold). Раз бот всегда ждёт, любое исходящее за окно = менеджер вручную — отдельная детекция «бот vs человек» не нужна. Рестарт переживается через startup-scan listen.

**Ночная тишина 22:00–08:00 по Амстердаму (безусловно).** Бот не отвечает ночью; отложенный ответ ждёт утреннего слота `next_morning_slot()` (рандом 08:15–09:30 = встроенный джиттер, чтобы утренняя пачка не ушла разом). `is_night_time()` (окно `NIGHT_START_AMS=22`/`NIGHT_END_AMS=8` в config). Старый env-флаг `BOT_REPLY_NIGHT_HOLD` удалён (был баг — `cap_bot_reply_delay` резал ожидание до 10 сек, ночью реально отвечал). overnight-сообщения бот разбирает сам после 08:00.

**Почему важно:** если кто-то спросит «почему бот отвечает медленно/через 30 мин» — это by design, а не баг. Регулируется env, без правки кода.

Параллельно усилена продающая часть: «подумаю / посоветуюсь / не сейчас» больше НЕ завершает диалог — мягкая лесенка 2-3 касания (признать паузу → открытый вопрос о сомнении → бесплатный необязательный шаг), с явными критериями стопа «чтобы не спугнуть». Правки в `knowledge/bot/tatiana_*.md` + `_CONTROL_TAIL` в llm_tatiana.py.

⚠️ ОБНОВЛЕНО 2026-05-29: прод-листенер БОЛЬШЕ НЕ на Mac. С 2026-05-27 мигрировал на Railway (`tg-outreach-listener`, workspace Womennetworking club) — см. [[wnw-outreach-railway-split]]. Локальные LaunchAgent'ы выгружены, процессов на Mac нет (проверено 29.05). Описание grace-механики ниже актуально, изменился только хостинг.

**How to apply:** После рестарта startup-scan не пропускается при grace>0 — проверять лог `🔁 listen_last_run пока не обновляю (отложено ответов: N)` на волну автоответов по старым тредам. Связано: [[feedback_wnw_outreach_strategy_april19]], [[feedback_wnw_outreach_rules]], [[feedback_no_answer_hints]].
