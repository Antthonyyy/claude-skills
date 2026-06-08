---
name: Telegram Outreach — документация
description: Полная документация по инструменту парсинга Telegram-групп и рассылки в личку (docs/telegram-outreach/)
type: reference
originSessionId: 98de5ce5-5b03-4ada-bc67-3eb0cb7579c1
---
Документация по инструменту Telegram Outreach находится в `docs/telegram-outreach/`:
- `README.md` — быстрый старт, команды
- `setup.md` — подготовка аккаунта (SIM, Premium, API, прогрев)
- `anti-ban.md` — безопасность, лимиты, правила анти-бана
- `workflow.md` — полный процесс от парсинга до рассылки
- `funnel.md` — воронка, шаблоны касаний 1-4, стоп-сигналы
- `architecture.md` — технические детали, файлы, форматы данных

Главный скрипт: `scripts/tg_outreach.py` (export, analyze-chat, send, status)
Сессия парсера: `tg_analyzer.session` (@AntonGgn)
Сессия рассылки: `tg_outreach.session` (TBD)
