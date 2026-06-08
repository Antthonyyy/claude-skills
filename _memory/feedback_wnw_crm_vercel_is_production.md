---
name: feedback-wnw-crm-vercel-is-production
description: Vercel-проект wnw-crm-app — это PRODUCTION CRM Татьяны (URL https://wnw-crm-app.vercel.app/). Любые preview-деплои в нём + любые правки env vars / редеплои = шевеление прода. Перед любыми write-действиями в этом проекте — явное подтверждение Антона.
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 48155be6-bb11-44e4-a515-477fdb97b2cc
---

**Production CRM Татьяны = Vercel-проект `wnw-crm-app` в аккаунте `gnilosiranton-8438s-projects`.**

- Production URL: `https://wnw-crm-app.vercel.app/`.
- Preview URL вида `wnw-crm-app-git-<branch-truncated>-<scope>.vercel.app` — **тот же проект**, другой deployment, **те же production env vars** при условии что scope включает Preview.
- Backend: prod-Supabase `uqclcbjfetjdplldevwv` (это БД клуба, не staging).
- Backend API: `wnw-api` на Railway (клиентский аккаунт `clubwomennetworking@gmail.com`, проект `a81373a6`).

**Why:** в сессии 2026-06-02 я принял preview deployment проекта `wnw-crm-app` за «безопасную песочницу» и:
- расширил scope 10 production env vars (включая `NEXTAUTH_SECRET`, `ADMIN_PASSWORD_HASH`, `SUPABASE_SERVICE_KEY`, `WNW_API_SECRET`, `UPSTASH_REDIS_REST_TOKEN`) на Preview через Vercel internal API — то есть выставил production-секреты на preview deployments (любой, кто получает preview URL и аутентифицирован в Vercel-аккаунте, попадает в prod CRM с prod-creds);
- запланировал smoke (Zoom CSV import + manual toggle attended) через preview-URL, не понимая, что preview-деплой бьёт в **prod БД и пишет в prod-таблицы Татьяны**, не в staging;
- ранее уже сеял тестовые контакты (`smoke_csv_test`, `smoke_manual_test`) прямо в `public.contacts` и `public.event_registrations` prod-Supabase, считая это «нормальной чисткой следом» — но это были записи в **прод-данные клуба**, а не в staging-сэндбоксе.

Признак заблуждения: дашборд Vercel показывал «No Production Deployment» в overview, но это либо дисплейный quirk, либо стейт для конкретного scope/team — production deployment у проекта **есть** (`target: production` в `/api/v6/deployments`), он живёт на `wnw-crm-app.vercel.app`. «No Production Deployment» в overview ≠ «это staging-проект».

**How to apply:**

1. Прежде чем трогать env vars, делать redeploy, патчить deployment-target в проекте `wnw-crm-app` — **явно подтверждать у Антона**, что понимаешь: это prod-проект Татьяны. Никаких «безопасных preview-экспериментов».
2. **Никогда не расширять Production-only env vars на Preview/Development scope без явного разрешения.** Production-секреты должны оставаться Production-only. Если для preview нужны env — это **отдельные**, минимально-привилегированные значения (staging Supabase, фейковые admin creds), не клон production.
3. Любой smoke / тестовый webhook / тестовый CSV / тестовая запись через preview URL = **запись в prod-БД клуба**. Если нужен real smoke без побочки — поднимать локально (`npm run dev` + `.env.local` с тестовыми creds), либо отдельный staging-проект, либо использовать чисто read-only проверки.
4. Перед любым `PATCH /api/v9/projects/wnw-crm-app/env/...`, `POST /api/v13/deployments` через browser-evaluate в Vercel — пауза + подтверждение. Они выглядят как «мелкие настройки», а по факту это правка прод-конфига.
5. Тестовые контакты в `public.contacts` / `public.event_registrations` prod-Supabase = **prod-мусор у Татьяны**. Если seed нужен — после smoke удалять в той же сессии явным DELETE с RETURNING-проверкой, не «потом».

Связано: [[feedback_endpoint_exists_vs_used_in_prod]], [[feedback_verification_discipline]], [[project_wnw_railway_migration_to_client]] (там тот же паттерн — prod на отдельном клиентском Railway-аккаунте, перед действием всегда проверять scope).
