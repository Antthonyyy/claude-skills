---
name: Railway autosync — push в main = автодеплой в прод
description: Railway деплоит из origin/main автоматически. Любой push на main = изменение в продакшене, перед пушем обязательная перепроверка.
type: feedback
originSessionId: 09441818-7f2e-419e-94d8-9257bd7a6a24
---

# Railway autosync — историческая запись (DEPRECATED 2026-04-24)

**ВНИМАНИЕ:** AutoSync с GitHub в Railway **отключён** 2026-04-24 по запросу Антона.
Актуальное правило — в [feedback_railway_manual_deploy_per_service.md](feedback_railway_manual_deploy_per_service.md): деплой только вручную через `railway redeploy`, только в затронутом сервисе.

Ниже — контекст почему AutoSync был проблемой (для понимания историческим).

Ранее Railway был настроен на autosync с `origin/main`: любой `git push origin main` должен был автоматически триггерить деплой в production.

**Why:** Антон явно попросил запомнить это после пуша рефакторинга outreach-бота — я не учёл что пуш в main уже означает живые изменения в проде (tg_outreach.py работает в Railway для WNW outreach, аналогично rawform-scheduler).

**How to apply:**

- Перед `git push origin main` — обязательно прогонять тесты локально (`pytest tests/`), smoke-проверять основные пути кода, syntax check.
- Если изменения в `scripts/tg_outreach.py`, `scripts/outreach_core/*`, schedulerах — это прод-боты, их поведение меняется СРАЗУ после пуша.
- Не делать опрометчивых `git push --force`, не пушить незавершённые рефакторинги.
- Если риск высокий — предлагать Антону делать пуш самостоятельно после ревью локального diff.
- Откат: `git revert <sha> && git push` — тоже автоматически откатывает прод.

Сервисы в Railway, которые подхватывают autosync:

- `rawform-scheduler` (проект `welcoming-purpose`) — автосинк Meta Ads → Google Sheets
- `wnw-api` (отдельный репо Antthonyyy/wnw-api) — не в этом проекте
- `wnw-cron` (отдельный репо Antthonyyy/wnw-cron) — не в этом проекте

**ВАЖНО:** Антон говорил, что Railway autosync иногда **НЕ срабатывает** — после пуша нужно перепроверять (через `railway status --json` смотреть latestDeployment.createdAt и сравнивать с моментом пуша). При этом перепроверка должна быть **read-only** — не отправлять тестовых сообщений в прод-боты, чтобы не создать дублей.

**Что НЕ в Railway** (работает локально на Mac):

- `scripts/tg_outreach.py listen` — WNW outreach listener (AI-ответы Татьяны). Запускается через `scripts/listen_wrapper.sh` на Mac. После push моих изменений в main — код в Railway не обновится (его там нет), Python-процесс на Mac держит старый код в памяти. Для применения изменений нужен рестарт процесса на Mac:
  - `ps aux | grep tg_outreach.py` → найти PID Python-процесса
  - `kill -TERM <PYTHON_PID>` → graceful shutdown (Telethon закроет сессию)
  - `listen_wrapper.sh` автоматически перезапустит новый процесс через 5-10 сек со свежим кодом
  - **Тайминг критичен:** не рестартить во время `python send ...` (параллельная рассылка) и когда есть свежие pending_replies (последняя запись в `outreach_logs/tg_pending_replies.jsonl`) — иначе риск дубля AI-ответа (Telegram перевышлет непрочитанное входящее после реконнекта).
