---
name: reference_wnw_smartsender_funnels
description: WNW SmartSender — карта воронок (funnel id / serviceKey) + баг ярлыка уведомления бизнес-клуба
metadata: 
  node_type: memory
  type: reference
  originSessionId: b784043b-6f62-46ee-9c27-6b904b602478
---

Карта воронок SmartSender WNW (project `zapusk-kluba-300953`). В URL `messenger.smartsender.com/funnels/<X>` виден **serviceKey**, не funnel id.

| funnel id | serviceKey | Назначение |
|---|---|---|
| 1331893 | 1333159 | нетворкинг — полная (ПЕРВАЯ регистрация) → webhook source `smartsender_networking_wednesday` |
| 1360217 | 1361534 | нетворкинг — упрощённая (ПОВТОР) → webhook source `smartsender_networking_repeat` |
| 1360409 | 1361726 | бизнес-клуб (каждую субботу) → webhook `register-business-club`, source `smartsender_business_breakfast_saturday` |

Все три шлют POST на `https://wnw-api-production.up.railway.app/smartsender/...` с заголовком `X-SmartSender-Secret`.

**«(сокращенная)» = ПОВТОРНАЯ регистрация на нетворкинг** (2+ раз). Уведомление бота корректное (подтвердил Антон 23.05). Воронка упрощённого повторного нетворкинга = **1360217** «запись на нетворкинг (каждую среду) (Упрощенная)».

**5 «регистраций» 21–22.05 (проверено по API + БД):** в НАШЕЙ БД все записаны как `breakfast_registered` (register-business-club, source business_breakfast_saturday). По воронкам/тегам они РАЗНЫЕ: Юлиана и Lubow — только бизнес-клуб (1360409), теги только «Регистрация на бизнес завтрак»; Галина и Балюк — и в нетворкинг-упрощённой (1360217), и в бизнес-клубе, теги «Повторный вход на нетворкинг» + «завтрак». **Ни один не первичный нетворкинг → лист «Новички» = 0 корректно.** Сверка «уведомление ↔ вебхук ↔ воронка» (почему все 5 легли как завтрак, хотя текст «нетворкинг сокращ.») — на стороне Антона (live SmartSender), не наш код.

Связано: [[project_wnw_bot_sheet_newcomers]] (лист «Новички» = первая регистрация по воронке 1331893).
