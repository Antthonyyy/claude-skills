---
name: reference-orchestrate-skill
description: "Скилл /orchestrate — дирижёр задач Антона (план→критика→исполнение→сверка), когда и как применять"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 67a40385-1ed7-4a26-bee7-a4a080f5da36
---

`/orchestrate` (`~/.claude/skills/orchestrate/SKILL.md`, создан 2026-06-02) — скилл-дирижёр для нетривиальных задач. Машина состояний: **0 CAPTURE** (задача + verifiable-критерии + заземление file:line) → **1 PLAN** (N шагов, каждый = действие+проверка, в TodoWrite) → **2 CRITIQUE** (субагент, fresh ctx, автономный red-team+pre-mortem+WebSearch, без вопросов → PASS/FAIL + список дыр) → **3 REVISE** (петля, макс 3 круга) → **ЧЕКПОЙНТ** (показать план Антону, 1 раз) → **4 EXECUTE** (по шагам в режиме Karpathy; gate после каждого: код→/code-reviewer+/verify, маркетинг→сверка с критерием; ретрай ≤2) → **5 FINAL** (субагент, fresh ctx, сверка всех критериев, verified vs believed-done) → отчёт в `tasks/`.

Развилки, которые Антон выбрал при сборке: охват **универсальный** (код+маркетинг), критик **автономный + WebSearch** (НЕ звать интерактивный /the-fool как есть — он стопает на AskUserQuestion), автономия **1 чекпойнт на утверждение плана**, дальше до конца сам.

**Декомпозиция (выбор B, 2026-06-02):** критик вынесен в отдельный скилл `/critique` (`~/.claude/skills/critique/SKILL.md`) — автономный неинтерактивный, PASS/FAIL + список дыр, reusable standalone («раскритикуй вот это»). /orchestrate в фазах 2 и 5 запускает `/critique` В СУБАГЕНТЕ (свежий контекст = непредвзятость). Планировщик/gate НЕ выносили — это feature-forge/Plan-агент и /code-reviewer+/verify (уже есть). Полный сплит (C) отвергнут: дублировал бы готовое и грузил контекст.

**Когда применять:** Антон даёт задачу и хочет «нормально по этапам». НЕ для однострочных правок/вопросов/чтения. Зашивает обязательные правила памяти: [[feedback_ground_recommendations_in_code]], [[feedback_subagents_may_simulate_tool_calls]], [[feedback_verification_discipline]]. Karpathy ([[CLAUDE.md]] дефолт) — в фазе исполнения.
