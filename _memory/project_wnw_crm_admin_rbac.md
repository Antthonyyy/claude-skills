---
name: project-wnw-crm-admin-rbac
description: "WNW CRM фича admin-users RBAC (owner/manager/viewer) реализована в ветке, НЕ задеплоена — ждёт OK Антона."
metadata: 
  node_type: memory
  type: project
  originSessionId: 2a81e3c1-27b3-4c55-b602-82691585b843
---

WNW CRM (`/Users/mac/Downloads/wnw crm/`): фича управления админами с ролями реализована в ветке **`feat/admin-users-rbac`** (коммит `bc237ef`), 2026-06-05. **Локальный коммит, НЕ запушено, НЕ задеплоено — ждёт явного green light Антона.**

**Что: ** таблица `admin_users` (роли owner/manager/viewer, session_version, must_change_password), DB-backed next-auth вместо ENV-одиночного админа, **ленивая миграция** env-админа в `admin_users` при первом логине (email Антон не передавал — берётся из Vercel env), ревокация сессий через session_version (ре-валидация в jwt-колбэке, TTL 30с), last-owner advisory-lock trigger, preview-login guard (`AUTH_DISABLE_PREVIEW_LOGIN` Preview-scope), audit viewer «кто что делал» (`audit_log.actor_id` FK), форс-смена пароля. Гейт typecheck+lint+build = 0. Прошло состязательное ревью (3 ревьюера), 4 дефекта закрыты (viewer in-handler requireWriter, валидный DUMMY_HASH, bumpSessionVersion error-propagation, teamMemberId UUID).

**Enforcement: ** owner-only на управление админами; viewer = read-only (`requireWriter` на 15 мутирующих роутах + Edge fast-path); **manager сохранил полный операционный доступ вкл. платежи** — намеренно не ограничивал (Галина=manager, сломало бы её).

**Открытый продуктовый вопрос (phase-2): ** ограничивать ли manager на платежи/возвраты? Если да — Галине нужен owner либо отдельная capability.

**Runbook выкатки (когда даст добро): ** migration → `AUTH_DISABLE_PREVIEW_LOGIN=1` только Preview-scope → ротация `NEXTAUTH_SECRET` → `vercel --prod` → первый логин текущими кредами (создаёт owner) → **только потом** убрать `ADMIN_EMAIL`/`ADMIN_PASSWORD_HASH`/`ADMIN_NAME` (убрать раньше при пустой таблице = lockout). ТЗ: `clients/wnw/tasks/crm_audit_and_roadmap_2026-06-05.md` §0.

Связано: [[project_wnw_crm_perf_messages_underfetch]] (перф-долг той же CRM), [[feedback_wnw_crm_vercel_is_production]], [[feedback_propose_dont_execute_until_ok]].
