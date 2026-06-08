---
name: WNW offline nudge — шаблон v12 и логика применения
description: v12_offline_nudge.txt — для авто-дожима registered-контактов, замолчавших без офлайн-инвайта
type: project
originSessionId: 60cb9c92-2f01-456f-8bbc-c1faed0a5cb6
---
Шаблон `projects/wnw/tasks/messages/v12_offline_nudge.txt` — базовый для крон-скрипта авто-дожима.

**Why:** люди на стадии `registered` (онлайн-нетворкинг) замолкают, офлайн не предлагался. Тёплый контакт, а не холодный — стоит дожать.

**Логика применения:**
- `registered` → v12 (фраза "помимо онлайн-встреч" уместна, они знают что есть онлайн)
- `in_dialog` без регистрации → отдельный вариант шаблона (позже), там "помимо онлайн" не точна
- Один раз на контакт — флаг `offline_nudge_sent` в `outreach.events`

**Крон:** запускать раз в день, фильтр: `stage = 'registered' AND set_at < now() - interval '3 days'` + нет входящих 3+ дня.
