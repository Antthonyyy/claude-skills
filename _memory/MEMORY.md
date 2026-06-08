# Память — Маркетинговый ассистент Антона

> Структура: общая преамбула (критические правила) → **Личное / Anton** → **Работа / клиенты**.
> Граница папок: личное = `personal/`, работа = `clients/`. Маршрутизация — `ROUTING.md` в репо.

## Критические правила работы

### Проверка дублей перед рассылкой (ОБЯЗАТЕЛЬНО)
Перед повторным запуском любого batch-скрипта: `ps aux | grep <script>` + проверить последние записи в outreach.messages. Пустой output-файл ≠ "не запустился". Подробнее: [feedback_no_duplicate_sends.md](feedback_no_duplicate_sends.md)

### Проверять артефакты субагентов (ОБЯЗАТЕЛЬНО)
Делегированный агент может вернуть транскрипт с «выполненными» Write/Bash, не выполнив их. После делегирования — `ls`/Read проверять файлы на диске самому. [feedback_subagents_may_simulate_tool_calls.md](feedback_subagents_may_simulate_tool_calls.md)

### Заземлять планы по коду (ОБЯЗАТЕЛЬНО)
Любой план/рекомендация должна быть проверена по реальному дереву (file:line) до публикации — Антон режет «висящие в воздухе». [feedback_ground_recommendations_in_code.md](feedback_ground_recommendations_in_code.md)

### Дисциплина верификации (ОБЯЗАТЕЛЬНО)
Перед заявлением «всё работает» в отчёте о миграции — pgrep -f по имени wrapper'а (не только Python), читать каждую строку crontab на предмет guard'а, делить отчёт на «verified» vs «believed done». [feedback_verification_discipline.md](feedback_verification_discipline.md)

### Не киллить рекламный креатив по одному окну (ОБЯЗАТЕЛЬНО)
Перед «выключить» креатив в Meta Ads — три окна (lifetime + понедельно + last7d), гейт выборки (<8–10 конверсий = WATCH), сравнение с собственным базлайном, confound-чек (дубли крео/пересечение аудиторий/зрелость атрибуции/зомби). Урок: хардкилл post 9 Rawform по шумной 7-дневке (5 чатов), откатили. Методология — скилл `/meta-ads-audit`. [feedback_no_kill_ad_on_small_window.md](feedback_no_kill_ad_on_small_window.md)

### Frontend gate ОБЯЗАН включать lint (ОБЯЗАТЕЛЬНО)
Для React/Next.js-задач gate = `typecheck` + **`lint`** + `build`, проверять exit codes явно. `next build` НЕ ловит `react-hooks/purity` (`Date.now()`/`Math.random()` в render). Урок: PR #2 WNW CRM 2026-06-03 — гонял только tsc+build, lint падал, Антон поймал. [feedback_frontend_gate_must_include_lint.md](feedback_frontend_gate_must_include_lint.md)

### Код есть ≠ работает на проде (ОБЯЗАТЕЛЬНО)
При заявлении «X уже реализовано / работает» проверять не только наличие endpoint/UI в коде, но и фактическое использование в проде: `audit_log`, count(*) вызовов, дата последнего реального (не тестового) вызова. Summary Explore-агента про «уже есть» — это code-level claim, не prod-status. [feedback_endpoint_exists_vs_used_in_prod.md](feedback_endpoint_exists_vs_used_in_prod.md)

### Vercel `wnw-crm-app` = PRODUCTION CRM Татьяны (ОБЯЗАТЕЛЬНО)
Production URL `https://wnw-crm-app.vercel.app/`, preview URL `wnw-crm-app-git-<branch>-...vercel.app` — **тот же проект**, общие env vars при включённом Preview scope, общая prod-Supabase (`uqclcbjfetjdplldevwv`). Никаких «безопасных preview-экспериментов»: env-патчи / редеплои / smoke через preview UI = шевеление прода. Production-секреты НЕ расширять на Preview без явного OK. [feedback_wnw_crm_vercel_is_production.md](feedback_wnw_crm_vercel_is_production.md)

### Дрейф из диагностики в деплой (ОБЯЗАТЕЛЬНО)
Когда сессия read-only-скилла переходит в правку кода + прод-деплой — пауза, переключиться в /orchestrate или /critique. Перед `railway up`: пересечь список путей deploy-скрипта с `git status` по тем же путям; грязно → собирать чистый build-context из `git show HEAD:`. [feedback_dont_drift_diagnose_to_deploy.md](feedback_dont_drift_diagnose_to_deploy.md)

### Аудит/спроектировать = только предлагать (ОБЯЗАТЕЛЬНО)
Задача «проанализируй / спроектируй / собери roadmap / ТЗ» — markdown-артефакт в `clients/имя/tasks/`, прод-код не трогать, миграции не применять, без зелёного света Антона не двигаться дальше. Урок 2026-06-05: Антон остановил оркестратор словами «без меня ничего не делай». [feedback_propose_dont_execute_until_ok.md](feedback_propose_dont_execute_until_ok.md)

### Не переключаться с рабочего аккаунта на личный без OK (ОБЯЗАТЕЛЬНО)
Выбор аккаунта/идентичности — решение Антона, не моё. Тупик на выбранном пути ≠ право фолбэкнуть на личный аккаунт (личный TG @AntonGgn, личный Railway, личный Gmail). Описать тупик + варианты + ждать OK. Урок 2026-06-07: я сам запустил login на @AntonGgn после тупика с @WomenNetworking. [feedback_no_switch_to_personal_account.md](feedback_no_switch_to_personal_account.md)

### Гигиена git-коммитов (ОБЯЗАТЕЛЬНО)
Никогда `git add -A`; стейдж явными путями → показать staged-набор → проверить, что чужой WIP/секреты/PII не попали; несвязанное (реорг) — на отдельную ветку; перед merge проверять divergence. `.env`/`credentials/**` зашифрованы git-crypt — не untrack'ать. [feedback_git_commit_hygiene.md](feedback_git_commit_hygiene.md)

### git worktree + git-crypt — smudge падает, workaround
`git worktree add` падает на `.env` smudge, потому что ключ git-crypt не копируется в linked worktree. Если зашифрованные файлы не нужны: `-c filter.git-crypt.smudge=cat -c filter.git-crypt.required=false` + `update-index --skip-worktree .env`. При ретрае без `-b` (ветка успевает создаться от первой упавшей попытки). [feedback_git_crypt_worktree_workaround.md](feedback_git_crypt_worktree_workaround.md)

### Факты о внешних API — перепроверять веб-поиском (ОБЯЗАТЕЛЬНО)
Прежде чем уверенно заявить «нельзя / только так / не поддерживается» про возможности платформы или API (Telegram, Meta, сторонние сервисы) — WebSearch по официальным докам, особенно когда собираюсь что-то отмести как невозможное. [feedback_verify_external_facts_via_web.md](feedback_verify_external_facts_via_web.md)

### Meta Ads — таймзону кабинета изменить нельзя
Таймзона рекламного кабинета задаётся при создании и не меняется — не предлагать клиенту «переключить TZ». Если нужна другая — только новый кабинет. [feedback_meta_ad_account_tz_immutable.md](feedback_meta_ad_account_tz_immutable.md)

### Не обещать SLA, которого нет
Бот не должен писать лиду «Татьяна ответит лично», если процесс этого не гарантирует. Лучше нейтральный «зафиксировала, пока — бесплатный шаг…». [feedback_no_promised_sla.md](feedback_no_promised_sla.md)

### Исследование перед стратегией (ОБЯЗАТЕЛЬНО)
Перед любой рекламной стратегией, воронкой, медиапланом — сначала запустить реальный анализ:
1. `scripts/fb_ad_search.sh` или `scripts/ad_pipeline.py` — конкуренты в Meta Ads
2. `scripts/reels_analyzer.py` — вирусный контент Instagram конкурентов
3. `scripts/nb_query.py` — запрос в NotebookLM по теме стратегии
4. `scripts/tg_channel_analyzer.py` — анализ Telegram (если релевантно)

Без этого шага стратегия не строится. Правило установлено 2026-02-28.

Исключения: срочные точечные задачи (текст, правки), технические задачи (код, баги).

### Стиль общения
- Прямой, деловой, без воды и реверансов
- Антон — технический специалист, базовых вещей не объяснять
- Всегда на русском (код и комментарии тоже)
- [Литературный русский для инструкций](feedback_literary_russian_for_instructions.md) — памятки/регламенты/документы: только литературный язык, единый регистр (по умолчанию на «вы»), без сленга и англицизмов

### Проверка текстов крео на правила Meta (ОБЯЗАТЕЛЬНО)
Перед финализацией любого рекламного текста — проверять на нарушения политики Meta.
Подробности: [feedback_meta_ads_policy_check.md](feedback_meta_ads_policy_check.md)

### Сохранение артефактов
- Все результаты работы → `clients/имя/tasks/` (или `personal/имя/tasks/`)
- Результаты исследований → `clients/имя/tasks/research_ДАТА.md`
- Стратегии → `clients/имя/tasks/strategy_ДАТА.md`

### Railway — ДВА аккаунта, прод WNW на КЛИЕНТСКОМ (ОБЯЗАТЕЛЬНО проверять перед деплоем)
Прод WNW (listener/cron) — на аккаунте клиентки `clubwomennetworking@gmail.com`, проект wnw-api `a81373a6`, listener service `1405d447`. Деплой ТОЛЬКО клиентским токеном (`RAILWAY_TOKEN=<client> railway up --service NAME --ci`). CLI по умолчанию залогинен в ЛИЧНЫЙ `gnilosiranton` → `railway list` показывает только `Antthonyyy's Projects` + старый личный `cd7c8bfe` (это НЕ прод клиентки). **Перед ЛЮБЫМ Railway-действием: `railway whoami` + `railway list` + `railway status`.** Детали: [Railway → клиент](project_wnw_railway_migration_to_client.md).

**Токен для wnw-api лежит в `.env` (git-crypt) как `RAILWAY_TOKEN_WNW_CLIENT` — не спрашивать Антона каждый раз.** [Railway WNW token in .env](reference_railway_wnw_token_in_env.md). Account-token (workspace GraphQL, опционально): [Railway клиентский account-token](reference_railway_client_account_token.md) (/tmp/rw_account_tok, session-only).

### Railway — ручной деплой per-service (autosync выключен)
- [Railway manual deploy](feedback_railway_manual_deploy_per_service.md) — autosync с GitHub OFF (2026-04-24). Деплой ТОЛЬКО затронутого сервиса через `railway redeploy`, другие не трогать.
- [Railway autosync — DEPRECATED](feedback_railway_autosync_push.md) — историческая запись, зачем autosync был выключен
- [Railway variables --set перезаписывает](feedback_railway_variables_set_overwrites.md) — для CSV-переменных (WNW_MANAGER_IDS) читать текущее значение ПЕРЕД set, иначе сотрёшь существующих

### Реорг репо: личное/работа + симлинки (2026-05-16)
`projects/` физически разнесён: `personal/anton`, `clients/wnw`, `clients/rawform`. `projects/*` остались back-compat симлинками. Новый код использует `scripts/paths.py::project_path()` / `intel_path()` / `list_projects()`, не хардкодит `projects/...`. Карта — `ROUTING.md`.

### Изоляция Telegram-аккаунтов (ОБЯЗАТЕЛЬНО)
WNW-outreach-аккаунт ≠ group-monitor-аккаунт, ни в одну сторону. Все Telethon-entrypoints проходят `scripts/telegram_identity.py::assert_expected_account` (`TG_OUTREACH_EXPECTED_USER_ID` / `GROUPMON_EXPECTED_USER_ID`). group_monitor НЕ читает `TG_STRING_SESSION`.

---

## Личное / Anton

### Пользователь
- [Часовой пояс Антона](user_timezone.md) — Киев, Europe/Kyiv
- [Voice Profile](user_voice_profile.md) — тон, фразы, запреты, структура текстов Антона
- [Anti-AI Writing Guide](user_anti_ai_guide.md) — чек-лист запрещённых AI-паттернов при написании текстов
- [Перевод английских сообщений](feedback_english_translation.md) — всегда добавлять перевод на русский при английских текстах для клиентов
- [Не подсказывать ответ клиенту](feedback_no_answer_hints.md) — открытые вопросы вместо "X или Y?"
- [Делать сразу + перепроверять историю](feedback_execute_and_verify.md) — если Антон сказал сделать, делать в том же ходу; перед утверждением «не отправлено» проверять БД широко (ILIKE-шаблоны + templates + окно 48–72ч)
- [Не создавать файлы без явной команды](feedback_dont_create_files_unilaterally.md) — одобрение текста ≠ команда записать файл; спрашивать перед созданием
- [Читать историю перед ответом](feedback_read_history_before_reply.md) — всегда читать полную переписку с контактом перед генерацией ответа

---

## Работа / клиенты

### Проекты — рекламные кабинеты
- [Rawform — Meta Ads аккаунт](project_rawform_ad_account.md) — Cheerfull, ID 10152409487740455 (мебель, Бали)
- [Rawform — структура бюджета + TZ](project_rawform_budget_structure.md) — lifetime budget (не daily), AUD, TZ America/Los_Angeles (Bali = +15h)
- [Rawform — Railway деплой](project_rawform_railway.md) — rawform-scheduler на Railway, автосинк Google Sheets
- [Rawform — Meta API: видео + Dev-mode блокер](project_rawform_meta_api_video_devmode.md) — app в Dev mode не даёт видео-creative (subcode 1885183); endpoint advideos; шаблон CBO/WhatsApp кампаний

### Инструменты
- [/orchestrate — дирижёр задач](reference_orchestrate_skill.md) — план→автокритика+WebSearch→исполнение по шагам (Karpathy)→финальная сверка; 1 чекпойнт на план. Критик вынесен в `/critique` (reusable, неинтерактивный, PASS/FAIL)
- [Telegram Outreach — документация](reference_telegram_outreach_docs.md) — парсинг групп + рассылка в личку (docs/telegram-outreach/)
- [@WomenNetworking — логин только через QR](reference_womennetworking_qr_login.md) — SMS/app-код этому аккаунту НЕ приходит (повторяется); заходить через qr_login + 2FA, отдельная сессия (не файл/string — AuthKeyDuplicated)
- [WNW outreach — SQLite long-term память](reference_outreach_sqlite.md) — `outreach_logs/outreach.db` для контекста бота, событий воронки, алертов менеджеру

### WNW — правила диалогов
- [WNW outreach — правила диалогов](feedback_wnw_outreach_rules.md) — только русский язык, скрипт для участниц других клубов, ссылка на бизнес-завтрак
- [WNW outreach — стратегия диалогов (апрель 2026)](feedback_wnw_outreach_strategy_april19.md) — direct invite в первом касании, тон на «вы», цены в промте (370/600/750/1100€), поддержка горячих лидов
- [WNW — окно приоритета менеджера + лесенка продаж](project_wnw_manager_grace.md) — бот откладывает ВСЕ ответы (30 мин обычные / 5 мин горячие), второй живой менеджер; «подумаю» больше не закрывает диалог. Прод — локальный LaunchAgent, не Railway

### WNW — мероприятия
- [WNW — субботний завтрак как 2-й бесплатный вход + day-of-week routing](project_wnw_saturday_breakfast_deploy.md) — deployed 2026-05-30; бот зовёт на ближайшее событие (ср 17:00 / сб 10:00); «первое бесплатно» = одно суммарно; название «завтрак» НЕ переименовано
- [WNW Networking Schedule](project_wnw_networking_schedule.md) — нетворкинг каждую среду 17:00 CET; ближайший 27.05.2026; дата в шаблонах рассылки протухает — проверять перед каждым send
- [WNW outreach — ежедневная рассылка 20/день](project_wnw_outreach_daily_send.md) — отправка пока руками (cron нет), cron заблокирован захардкоженной датой в шаблонах; запускать send с `python -u`

### WNW — архитектура бота
- [WNW бот — архитектура промта (двухслойная)](project_wnw_bot_prompt_architecture.md) — поведение в код-листенере + промте + монолит-fallback; карта файлов для любой правки бота
- [bot-out template инвариант](project_wnw_bot_out_template_invariant.md) — любой bot-send в messages ОБЯЗАН нести template, иначе ломается F2-атрибуция бот↔менеджер (ложный manual_hold)
- [Failure-mode фиксы + деплой 2026-05-31](project_wnw_failure_mode_fixes_deploy.md) — 6 pre-mortem фиксов (commit 1bb8c8b5) на проде; listener service id 1405d447-1219...; грабли деплоя project-токеном (whoami не работает, stash WIP перед up)
- [Режим «менеджер» — коуч + whitelist](project_wnw_manager_assistant.md) — внутренний помощник менеджеров; whitelist WNW_MANAGER_IDS (Татьяна+Галина) только на Railway; tatiana_manager.md = коуч-плейбук (deploy 2026-06-02 f4aeeac8); карта файлов режима
- [Anti-fab deploy + 24h-check 2026-06-04](project_wnw_manager_bot_anti_fab_24h_check.md) — @assistant_wnw_bot фикс «ты»/«чат клуба»/кейс 9 (commit 4ae18bff, deploy 821eb02d, LaunchAgent fires 2026-06-05 09:30 Kyiv); rollback ручной

### WNW — payment intake (платящие из TG-групп → CRM)
- [Payment intake pipeline](project_wnw_payment_intake_pipeline.md) — ingestion новых членов клуба из TG-чата «Новенькие» в CRM (membership/subscription, two-step candidate→admin-confirm, endpoint /intake/club-member, manual-only v1). P1 (extractor) ГОТОВ; дальше P2 intake-таблица.

### WNW — тестирование
- [WNW промт — фаза тестирования](project_wnw_prompt_testing.md) — критерии мониторинга после внедрения продающей ветки (28.04.2026)

### WNW — технический долг
- [offline-wave bio_category](project_offline_wave_bio_category.md) — build_target_list() не тянет bio_category, generic-хук; fix: один SELECT + targets.append()
- [offline nudge шаблон v12](project_wnw_offline_nudge_template.md) — v12_offline_nudge.txt для registered-контактов (3+ дней тишины); in_dialog — отдельный шаблон позже
- [post-NW followup без проверки посещения](project_wnw_post_nw_followup_attendance_gap.md) — «как прошло?» шлётся по got_link отказникам; нет attendance-гейта (кейс 2026-06-04)
- [CRM perf + messages underfetch](project_wnw_crm_perf_messages_underfetch.md) — CRM грузит всю базу+обогащение в память на каждый рендер; cap PostgREST 1000 → messages недобираются (баг lastMessageAt/hasUnreadReply); план фиксов в crm_performance_analysis_2026-06-04.md
- [CRM admin RBAC (ветка, не задеплоено)](project_wnw_crm_admin_rbac.md) — фича admin_users (owner/manager/viewer) в ветке feat/admin-users-rbac (181a1f8), ленивый bootstrap, ревокация сессий, audit viewer; гейт зелёный, ревью пройдено; ждёт OK Антона на выкатку; phase-2 вопрос про manager/платежи

### WNW — WhatsApp Cloud API
- [WA Cloud Webhook статус](project_wa_cloud_webhook_status.md) — инфраструктура готова (Railway, webhook, ENV), блокер: OTP rate limit до 14:25 Kyiv 5 мая; Phone Number ID 109117..., WABA 204851...

### WNW — Google-таблица «бот» Татьяны
- [Таблица «бот» — доступ writer + лист «Новички»](project_wnw_bot_sheet_newcomers.md) — мы writer (не read-only); лист «Новички — ближайший нетворкинг» с дебютантами, cron sync_networking_newcomers.py, самоочистка по катящемуся окну
- [SmartSender — карта воронок WNW](reference_wnw_smartsender_funnels.md) — funnel id/serviceKey: нетворкинг полная/упрощённая/бизнес-клуб; баг ярлыка уведомления бизнес-клуба (пишет «нетворкинг», а это бизнес-завтрак)
- [Рассылка 2026 — крон + автоскрытие прошедших ивентов](project_wnw_outreach_sheet_2026.md) — sheet ID `1XJh...AheM`, cron `15 * * * *` (export_views_to_sheets + hide_past_event_sheets), v_* листы скрыты, hidden не сбрасывается

### WNW — архитектура и миграция
- [WNW CRM migration — outreach ↔ public через dual-write](project_wnw_crm_migration.md) — 21.04 смигрировали 161 контактов в public.contacts, dual-write через SAVEPOINT, зоны ответственности чатов разделены, RLS включён
- [WNW Cron — незаданные env vars](project_wnw_cron_missing_env.md) — SMARTSENDER_FUNNEL_DOGON + MANAGER_TELEGRAM_* не заданы на Railway, Антон просил напомнить потом
- [WNW Railway → аккаунт клиентки (2026-05-26)](project_wnw_railway_migration_to_client.md) — wnw-api/wnw-cron переехали на Womennetworking club (Hobby), новый URL `wnw-api-production-ae17.up.railway.app`, деплой через project tokens
- [WNW outreach на Railway — split TG/non-TG (2026-05-27)](project_wnw_outreach_railway_split.md) — listener держит ВСЕ TG-задачи (listen + 4 send-job'а в одном контейнере), cron — только non-TG. Иначе AuthKeyDuplicated.
