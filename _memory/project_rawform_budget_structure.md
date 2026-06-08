---
name: project-rawform-budget-structure
description: "Rawform Meta Ads — кампании на lifetime budget, не daily; TZ кабинета America/Los_Angeles"
metadata: 
  node_type: memory
  type: project
  originSessionId: 686e37d1-2e66-4067-8d2b-6b0fa7e2b387
---

Кабинет Rawform **Cheerfull** (`act_10152409487740455`) работает на **lifetime budget** (бюджет на весь период кампании), не на daily. Все 4 кампании, что были активны в мае 2026, — lifetime, диапазон 135–220 AUD на ~11–24 дня каждая. Валюта аккаунта — **AUD**.

**Таймзона кабинета — America/Los_Angeles** (UTC-7). Бали — UTC+8. Сдвиг: LA 00:00 = Bali 15:00. Все hourly/daily отчёты из API приходят в LA-времени, для клиента надо конвертировать в Bali. Поменять TZ кабинета нельзя ([feedback-meta-ad-account-tz-immutable]).

**Why:** Антон уточнил 2026-05-30. Я ошибочно отвечал клиенту про «дневной бюджет», хотя у нас lifetime. Это меняет всю терминологию: «когда заканчивается дневной бюджет» — некорректный вопрос, заканчивается общий бюджет к stop_time кампании.

**How to apply:**
- При обсуждении бюджета с Онуром — всегда сначала смотреть, какой режим стоит (daily/lifetime) через `/campaigns?fields=daily_budget,lifetime_budget,budget_remaining,start_time,stop_time`.
- При hourly-аналитике конвертировать LA → Bali (+15 часов) перед показом клиенту.
- При вопросе «удвоить бюджет» — варианты: (а) увеличить lifetime текущей кампании, (б) запустить новую с x2, (в) дублировать ad set. Просто «поднять daily» нельзя.
- Перед любым отчётом проверять `budget_remaining` и `stop_time` — кампании заканчиваются, надо вовремя продлевать.

См. также: [project-rawform-ad-account].
