---
name: project_wnw_post_nw_followup_attendance_gap
description: WNW post-NW follow-up шлёт «как прошло?» по got_link без проверки посещения — попадает на отказников
metadata: 
  node_type: memory
  type: project
  originSessionId: fc26d243-ef37-489e-af6f-90213d4eafc7
---

Пост-нетворкинговое касание (`run_follow_ups --type attended` → `find_eligible_post_nw_followup` в `scripts/outreach_db.py:1405`) отбирает когорту по `got_link` в окне `[nw−7д; nw+2ч]` и **не проверяет фактическое посещение** (system attendance не трекает, стадии дальше `got_link` нет).

**Why:** исключения query (`we_wrote_after`/`they_wrote_after`) считаются от `nw_at = 17:00`, поэтому дорегистрационные отказы (утром перед событием) НЕ отсекают контакт, а простой «не приду» не триггерит `got_link_rolled_back`. Результат: «как впечатления от среды?» уходит людям, которые явно сказали, что не будут.

Кейс 2026-06-04: из 5 получателей post_networking_check 3 явно отказались от события 03.06 (Zhanna→10.06, @womanskill, @MiraSwami→завтрак 06.06), 1 молчал, лишь @LevkovichMaryia подтвердила. Продажа не сдвинулась — все 5 остались на `got_link`, 4/5 не ответили. Подробно: `clients/wnw/tasks/post_nw_sales_check_2026-06-04.md`.

**How to apply:** при правке post-NW логики — добавить гейт посещения/отказа: исключать контактов с детектом «не приду / на след. неделе / сегодня не смогу» по ЭТОМУ событию; для перенёсшихся слать «ждём на ближайшей», а не «как прошло». Связано с [[project_offline_wave_bio_category]].
