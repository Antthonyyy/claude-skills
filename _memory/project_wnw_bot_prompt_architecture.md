---
name: project_wnw_bot_prompt_architecture
description: "WNW ИИ-продавец — поведение в ДВУХ слоях (код-листенер + промт), карта файлов для любой правки"
metadata: 
  node_type: memory
  type: project
  originSessionId: ab228724-7e4e-4f41-9d1f-611cb9b9538f
---

Поведение бота-продавца WNW задаётся в **двух слоях** — критично понимать перед любой правкой, иначе чинишь не там.

**Слой 1 — код-листенер `scripts/tg_outreach.py`:** детерминированные хард-стопы ДО вызова LLM.
- `_is_pure_gratitude` (~640) — глушит чистую благодарность.
- `is_telegram_auto_reply` (~617) — пропускает автоответчики.
- `manual_handoff.classify_manual_handoff` — эскалация на человека по B2B/founder/partnership/event-триггерам.
- Грейс-окно менеджера (отложенные ответы).
- **Важно:** эти проверки идут раньше LLM (`if _is_pure_gratitude(...): return` etc.). Если такой код-suppressor молча гасит — LLM никогда не увидит сообщение, никакой промт не поможет. Сначала всегда проверять `/tmp/listen_out.txt` на «бот молчит» строки.

**Слой 2 — промт `scripts/outreach_core/llm_tatiana.py`:**
- `build_system_prompt(stage=...)` собирает `core + stage_module + _CONTROL_TAIL`.
- `_select_stage_module` маршрутизирует по стадии воронки: `not_replied/sent_first_touch → cold`, `in_dialog/got_link/registered → dialog`, `attended/qualified → warm`, `customer → customer`, `declined/manual_hold → ""` (без модуля).
- `_CONTROL_TAIL` (~96-103) — последний и **самый сильный** блок, доминирует над модулями. Если правишь поведение, которое должно соблюдаться всегда — правь здесь, не в модулях.
- При `not _modules_ready()` → fallback на монолит `tatiana_system.md` (DEPRECATED, не зеркалится, при срабатывании один раз предупреждает в stdout).

**Карта файлов промта (`knowledge/bot/`):**
- `tatiana_core.md` — персона, общие правила, доверие, спец-случаи. Грузится для всех стадий.
- `tatiana_cold.md` — первое касание + холодные цены.
- `tatiana_dialog.md` — активный диалог (in_dialog/got_link/registered) + возражения.
- `tatiana_warm.md` — тёплая продажа после нетворкинга.
- `tatiana_customer.md` — оплатившая участница.
- `tatiana_offline.md` — офлайн-ланчи, Берлин VIP.
- `tatiana_system.md` — DEPRECATED монолит (fallback, не источник правды).

**How to apply:**
1. **Перед любой правкой поведения** — определить слой: код-suppressor (молчит/эскалирует) или промт-формулировка (что отвечает). Симптом «бот молчит» → код. Симптом «бот отвечает неправильно» → промт.
2. **Шаблоны first-touch** (`clients/wnw/tasks/messages/v*.txt`) — для рассылки, плейсхолдеры `{name_prefix}` (новое имя-контракт), `{first_name}` (legacy), `{bio_phrase}`, `{next_wed}`. Рендерятся в `cmd_send`. Не путать с промтом — это разные пути.
3. **Деплой правок промта/кода** — рестарт LaunchAgent `com.anton.tg-outreach.listener` (модули грузятся при импорте). Шаблоны и `cmd_send` подхватываются ежедневным cron-ом автоматически (fresh-read per запуск).
4. **Тесты** — `tests/test_pure_gratitude.py`, `test_autoreply_detect.py`, `test_name_salutation.py`, `test_timezone_line.py`, `test_prompt_modules.py`, `test_system_prompt_assembly.py`. После любой правки — `pytest tests/ -q`. Связано с [[project_wnw_outreach_daily_send]].
