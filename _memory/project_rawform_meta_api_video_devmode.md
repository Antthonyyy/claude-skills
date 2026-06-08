---
name: project_rawform_meta_api_video_devmode
description: "Rawform Meta Ads API — видео-creative нельзя создать (app в Dev mode), endpoint видео, шаблон кампаний"
metadata: 
  node_type: memory
  type: project
  originSessionId: 7804bb87-95b9-4142-b5d7-75109470c963
---

Создание/правка кампаний Rawform через Graph API (кабинет Cheerfull `act_10152409487740455`, AUD, TZ America/Los_Angeles).

**Блокер видео-креативов:** app `1248818236646154` («Targetolog AI Assistant», и `META_ACCESS_TOKEN`, и `FB_ACCESS_TOKEN` — один app) в **Dev mode**. Создание видео ad creative падает `error_subcode 1885183` («оформление создано приложением в режиме разработки») — и для page-only, и для page+IG. Кодом не обойти. Существующие объявления делались в **Ads Manager** (first-party app Meta). Фикс: перевести app в Live (developers.facebook.com), либо доделывать creative+ad в Ads Manager. Кампанию и adset API создаёт нормально (PAUSED), только creative блокирован.

**Загрузка видео:** endpoint `POST https://graph-video.facebook.com/v21.0/{account}/advideos` (множественное число `advideos`, НЕ `advideo`; subdomain `graph-video`). Потом poll `GET {video_id}?fields=status` до `status.video_status=="ready"`, тумбнейл — `GET {video_id}/thumbnails`.

**appsecret_proof:** токен и `META_APP_SECRET` из разных пар → proof невалиден (400). Read и write работают **без** appsecret_proof (у app выключен require-proof).

**Шаблон рабочих кампаний (повторять при создании):** objective `OUTCOME_ENGAGEMENT`, **lifetime-бюджет (CBO) на кампании** в minor units AUD (НЕ daily, НЕ на adset), `bid_strategy LOWEST_COST_WITHOUT_CAP`. Adset: `optimization_goal CONVERSATIONS`, `billing_event IMPRESSIONS`, `destination_type WHATSAPP`, `promoted_object {page_id 1054360961089470, whatsapp_phone_number 6282135767870}`, age 30–60, `advantage_audience 0` (OFF), плейсменты instagram (stream/story/reels) + whatsapp. Creative: `object_story_spec` с page_id `1054360961089470` + instagram_user_id `17841480175979423` + `video_data` (CTA `WHATSAPP_MESSAGE`). График: stop_time у всех ~`14:47 LA`.

`scripts/meta_executor.py` умеет только image-креативы (`SINGLE_IMAGE`) и дефолтит `INSTAGRAM_DIRECT` — для этого кабинета неверно (нужен WhatsApp). Кастомный скрипт: `scripts/rawform_create_contractors.py` (идемпотентный via env `RAWFORM_VIDEO_ID/CAMPAIGN_ID/ADSET_ID`). См. [[project_rawform_ad_account]], [[project_rawform_budget_structure]].
