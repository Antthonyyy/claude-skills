---
name: project-wnw-manager-assistant
description: "WNW-бот режим «менеджер» — внутренний коуч; whitelist на проде, файл-плейбук, карта файлов"
metadata: 
  node_type: memory
  type: project
  originSessionId: 67a40385-1ed7-4a26-bee7-a4a080f5da36
---

WNW-бот (аккаунт Татьяны `@Tatiana_Zakharov`, id 8442538511) имеет **режим «менеджер»** — внутренний помощник-коуч для менеджеров клуба (отвечает по клубу и продажам, НЕ продаёт менеджеру). Опознание по whitelist `WNW_MANAGER_IDS`; менеджерский путь изолирован от лид-воронки (ничего не пишет в БД/Sheets).

**Whitelist на проде (2026-06-03):** `WNW_MANAGER_IDS = 652907258,@tatyanaclub,950531308,@Galina_Poltavskay,587452006,@AntonGgn,8442538511,@Tatiana_Zakharov` — Татьяна (два её аккаунта: `@tatyanaclub` 652907258 + основной менеджерский `@Tatiana_Zakharov` 8442538511, тот же id что и outreach-сессия листенера, но именно с него Татьяна РУЧНО задаёт боту вопросы), Галина, Антон. Задаётся на Railway env **обоих** сервисов (`tg-outreach-listener` + `tg-manager-bot`), битово равны; в `.env` НЕТ. Числовой id приоритетнее username ([config.py:is_manager]). Добавлять менеджеров — merged-set (см. [[feedback_railway_variables_set_overwrites]]).

**Реальная Татьяна = id 8442538511 (@Tatiana_Zakharov), а не @tatyanaclub.** `@tatyanaclub` (652907258) — её отдельный клубный аккаунт, оставлен в whitelist на всякий случай, но основной канал коуч-сессий — `@Tatiana_Zakharov`. В прошлой версии памяти этого знания не было — `8442538511` ошибочно ассоциировался только с outreach-листенером.

**Карта файлов режима:** роутинг `tg_outreach.py:3051-3053` (is_manager→`_handle_manager_message:2930-2996`, ранний return до воронки); опознание `outreach_core/config.py:134-163`; промт `outreach_core/llm_tatiana.py:417-443` (`build_manager_system_prompt` = header + core + warm + override-tail, модель claude-sonnet-4-6); знания `knowledge/bot/tatiana_manager.md`; тесты `tests/test_manager_mode.py` (16, fitness-тест изоляции воронки). Файл читается 1 раз на импорте → правка требует redeploy listener.

**Плейбук (2026-06-02, commit f4aeeac8):** `tatiana_manager.md` прокачан из голой роли в коуч-плейбук — МЕТОД [стадия→причина→шаг→готовая реплика] + библиотека 8 ситуаций (вкл. «молчит после цены/реквизитов»). Зашиты: предохранитель от выдумки (нет факта→к Татьяне), развилка по температуре лида (после цены не откатывать на «бесплатно»), SLA-caveat («в течение часа» не гарантия), anti-capitulation на soft-objection. Факты не дублированы — деферятся в core+warm. Прогон через `/orchestrate`, отчёт `clients/wnw/tasks/orchestrate_manager_assistant_2026-06-02.md`.

**Деплой manager-bot — ОТДЕЛЬНЫЙ сервис (уточнено 2026-06-03):** прод-коуч — это НЕ in-listener путь, а отдельный **aiogram Bot API** бот `@assistant_wnw_bot` (id 8955973549, токен ≠ Telethon-сессии → AuthKeyDuplicated не грозит). Исходник на ветке **`feat/wnw-manager-bot`** в worktree **`/Users/mac/targetolog-manager-bot`** (commits d0bcb04b + 47aca259): `scripts/tg_manager_bot.py` (aiogram polling), `scripts/outreach_core/manager_bot_db.py`, `Dockerfile.manager-bot` (COPY outreach_core + tg_manager_bot.py + telegram_identity.py + **knowledge/bot/** + requirements.manager-bot.txt; CMD `python -u scripts/tg_manager_bot.py`), multi-service `railway.toml`. Деплой из worktree: `RAILWAY_TOKEN=$RAILWAY_TOKEN_WNW_CLIENT railway up --service tg-manager-bot --ci`. Верификация: лог `🧑‍💼 whitelist…` + `identity-guard OK @assistant_wnw_bot` + `aiogram… Start polling`.

**⚠️ ПРАВИЛО: правка `knowledge/bot/tatiana_warm.md` или `tatiana_system.md` требует редеплоя ДВУХ сервисов** — `tg-outreach-listener` (Dockerfile.listener, ветка saturday-breakfast) И `tg-manager-bot` (Dockerfile.manager-bot, ветка/worktree manager-bot), т.к. `build_manager_system_prompt = header+core+**warm**+tail`. Файлы на ветках расходятся → принести правку в обе; на manager-ветке `git checkout <src-branch> -- <files>` безопасен, если она не дивергировала (проверять diff к pre-base). Прецедент 2026-06-03: оффер «ценность клуба, не нетворкинг» — listener commit `555f3d9e`/deploy `51773468`, manager-bot commit `5f91fd56`/deploy `bac5f648`. См. [[feedback_railway_manual_deploy_per_service]], [[reference_railway_wnw_token_in_env]].

**Остаточно:** warm.md:23 всё ещё держит SLA «напишет в течение часа» (плейбук переопределяет текстом; жёсткая зачистка трогает лид-режим — отдельное решение). Acceptance-проба: менеджер пишет тест → лог `🧑‍💼 МЕНЕДЖЕР` + ответ-помощник.
