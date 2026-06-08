---
name: project-wnw-manager-bot-anti-fab-24h-check
description: WNW @assistant_wnw_bot — деплой anti-fab плейбука 2026-06-04 + LaunchAgent 24h-check на 2026-06-05 09:30 Kyiv; ручной rollback по решению Антона
metadata: 
  node_type: memory
  type: project
  originSessionId: 931d42ae-8378-4b26-8e65-a4bae46ec58d
---

Деплой плейбука `tatiana_manager.md` против выдумок (А1 «ты» / А2 «чат клуба») и кейса 9 — на проде с 2026-06-04 09:28 Kyiv.

**Why:** 24-часовое окно наблюдения за регрессом перед признанием фикса стабильным. Решение по rollback Антон принимает руками — авто-rollback в скрипте явно не активирован.

**How to apply:**
- Если в новой сессии Антон упомянёт «24h-check» / «manager-bot фикс» / «как там тон» — подтянуть факты ниже, не перепроверять с нуля.
- При обнаружении нарушений (`api_errors>0`, `pct_dont_know>25%` при `total_bot≥8`, или тон/выдумка в текстах) — **не запускать `git revert` сам**, сообщить Антону и ждать явной команды.
- B (листенер) и `MANAGER_CLAUDE_MODEL` отдельная переменная — отложенные follow-up, не делать без запроса.

## Факты для подтягивания

| Поле | Значение |
|---|---|
| Сервис | `tg-manager-bot` (Railway клиентский `wnw-api` `a81373a6`) |
| Бот | `@assistant_wnw_bot` (id `8955973549`) |
| Модель env | `CLAUDE_MODEL=claude-haiku-4-5-20251001` (Haiku оптимален по Anthropic well-being benchmark) |
| Deploy id | `821eb02d-e530-47bb-a388-cdcf7c986716` (SUCCESS 2026-06-04 09:28 Kyiv) |
| Pre-deploy SHA (rollback target) | `5f91fd5643991d1e885de5d9112fb86615e9bf3a` |
| Deploy commit | `4ae18bff` в `feat/wnw-manager-bot` (cherry-pick из `d7058110` в `fix/wnw-manager-prompt-anti-fabrication`) |
| Worktree manager-bot | `/Users/mac/targetolog-manager-bot` (ветка `feat/wnw-manager-bot`) |
| PR fix→main | НЕ открыт; ветка `fix/wnw-manager-prompt-anti-fabrication` запушена в origin, ждёт PR |
| 24h-check trigger | LaunchAgent `com.wnw.manager-bot-24h-check`, plist в `~/Library/LaunchAgents/`, fires 2026-06-05 09:30 Kyiv |
| Скрипт проверки | `outreach_logs/manager_bot_24h_check_2026-06-05.py` (date-guard на 2026-06-05) |
| Отчёт после прогона | `clients/wnw/tasks/orchestrate_manager_prompt_anti_fab_24h_check_2026-06-05.md` |
| Связанные отчёты | `analysis_assistant_wnw_bot_2026-06-03.md`, `orchestrate_manager_prompt_anti_fab_2026-06-04.md` |

## Критерии Антона на 24h срез

- `api_errors = 0` иначе — разбор `railway logs --service tg-manager-bot --since 24h`
- `pct_dont_know ≤ 25%` (порог rollback)
- `total_bot < 8` → процент не считать rollback-сигналом, смотреть тексты вручную
- Тексты post-deploy не содержат: запрещённых форм 2 л. ед. ч. («ты прав / уточни / посмотри / у тебя / ...»), выдуманного «чата клуба» для не-членов

## Rollback (ручной, по команде Антона)

```bash
cd /Users/mac/targetolog-manager-bot
git revert HEAD --no-edit
git push
export RAILWAY_TOKEN="$(grep -m1 '^RAILWAY_TOKEN_WNW_CLIENT=' /Users/mac/targetolog-ai-assistant/.env | cut -d= -f2-)"
railway up --service tg-manager-bot --ci
```

## Cleanup после успеха

```bash
launchctl bootout gui/$(id -u)/com.wnw.manager-bot-24h-check
rm ~/Library/LaunchAgents/com.wnw.manager-bot-24h-check.plist
```

Связано: [[project-wnw-manager-assistant]] (исходный whitelist + плейбук), [[feedback-verification-discipline]] (verified vs believed), [[feedback-wnw-crm-vercel-is-production]] (общий принцип ручного подтверждения прод-операций).
