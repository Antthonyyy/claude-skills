---
name: WA Cloud Webhook — статус и блокер
description: WhatsApp Cloud API webhook для WNW speakers. Инфраструктура готова, блокер — OTP rate limit на номере +380 97 994 6252
type: project
originSessionId: 543a939f-a008-47f1-9fa5-8d7e09f9dfc3
---
Всё настроено, один блокер: OTP rate limit Meta на номере +380 97 994 6252.

**Why:** OTP через API запрашивался дважды (SMS + VOICE), Meta вернул `error_subcode: 2388367` — "слишком много запросов". Rate limit per phone number, сбрасывается ~24ч с последнего запроса (~14:25 Kyiv, 4 мая → сбрасывается 5 мая после 14:25 Kyiv).

**How to apply:** При следующем разговоре — сразу идти на OTP-запрос через API, не через UI. После получения кода от Антона — verify_code + register.

## Готовая инфраструктура

- Railway service: `wa-cloud-webhook` (welcoming-purpose project, production env)
- URL: https://wa-cloud-webhook-production.up.railway.app
- Health: /health → 200 OK
- Webhook: зарегистрирован, active:true, field messages, GET verification прошла

Railway ENV (все 10 выставлены):
- OUTREACH_BACKEND=postgres
- SUPABASE_DB_URL ✅
- WA_VERIFY_TOKEN=DdcceDX2LOlQrVjN25KsvogmeaX_HVzt-emXBsq3lzg
- WA_REQUIRE_SIGNATURE=1
- WA_APP_SECRET ✅ (dee28008...)
- WA_ACCESS_TOKEN ✅ (60-дневный, до ~3 июля 2026, WA permissions granted)
- WA_PHONE_NUMBER_ID=1091179940748252 (+380 97 994 6252)
- WA_GRAPH_API_VERSION=v22.0
- TELEGRAM_BOT_TOKEN ✅
- WNW_LEADS_NOTIFY_CHAT_ID=587452006

Meta:
- App: "Targetolog AI Assistant" (ID: 1248818236646154)
- WABA: "Tatiana Zakharova" (ID: 2048515475727437) в WNW business (1036271991958769)
- Phone Number ID: 1091179940748252
- App подписан на WABA: /subscribed_apps → success
- Webhook subscription: active, object=whatsapp_business_account, field=messages

## Правила OTP и PIN (зафиксированы Антоном 2026-05-04)

- Завтра после 14:25 Kyiv — ТОЛЬКО ОДИН SMS `request_code`. VOICE не делать. Если SMS нет 10 мин — остановиться, спросить Антона.
- Первый ответ бота в E2E: AI/GDPR-дисклеймер + "Здорово. Как к вам обращаться?"
- external_id/phone в Supabase = номер клиента (кто написал), не бизнес-номер
- E2E проходить до `state=done` (все 5 шагов формы)
- Ads Manager не трогать до успешного E2E; live ad sets не редактировать

**PIN для /register: `602101` (временный — был показан в чате)**
- Сохранить в Bitwarden: "WNW WhatsApp Cloud API — register PIN", password=602101
- После /register: если WhatsApp Manager позволяет сменить two-step PIN → сгенерировать новый, сохранить только в Bitwarden, написать "PIN rotated and saved to Bitwarden" без значения
- Если ротация невозможна сразу → написать "PIN rotation pending" + где сделать вручную

## Следующие шаги (после 14:25 Kyiv 5 мая)

1. Запросить SMS (один запрос!):
```bash
curl -s -X POST "https://graph.facebook.com/v22.0/1091179940748252/request_code" \
  -H "Authorization: Bearer <WA_ACCESS_TOKEN из Railway>" \
  -H "Content-Type: application/json" \
  -d '{"code_method":"SMS","language":"uk"}'
```

2. Попросить Антона прислать код из SMS на +380 97 994 6252.

3. Верифицировать:
```bash
curl -X POST "https://graph.facebook.com/v22.0/1091179940748252/verify_code" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"code":"XXXXXX"}'
```

4. Зарегистрировать для Cloud API (PIN произвольный 6 цифр, запомнить):
```bash
curl -X POST "https://graph.facebook.com/v22.0/1091179940748252/register" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"messaging_product":"whatsapp","pin":"XXXXXX"}'
```

⚠️ После /register — WhatsApp Business App на телефоне отключится (ожидаемо, Антон подтвердил).

5. E2E тест: написать на +380 97 994 6252 "Хочу подать заявку на спикерство WNW."
6. Проверить Supabase outreach.wa_applications (funnel=speakers, state=done)
7. Проверить TG alert в chat_id 587452006

## Что осталось после регистрации

- Click-to-WhatsApp ads в Ads Manager: prefilled "Хочу подать заявку на спикерство WNW."
- Возможно: добавить способ оплаты в WABA (для business-initiated messages, для customer-initiated не нужно)
- WA_ACCESS_TOKEN истекает ~3 июля 2026 — обновить заранее
