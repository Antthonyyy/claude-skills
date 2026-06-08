---
name: feedback_frontend_gate_must_include_lint
description: "Для frontend/Next.js-задач gate ОБЯЗАН включать npm run lint, не только typecheck+build — next build не ловит react-hooks/purity"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 09dc5bb6-43fa-425a-9fea-2f2dfe44f605
---

Перед заявлением «готово/PASS» на frontend-задаче (React/Next.js) gate обязан включать **`npm run lint`** (eslint), а не только `tsc --noEmit` + `next build`.

**Почему:** `next build` по умолчанию НЕ прогоняет eslint с правилами вроде `react-hooks/purity`. На PR #2 WNW CRM (2026-06-03) я прогнал tsc + build (оба PASS), заявил готовность — а `npm run lint` падал на `Date.now()`, вызванном прямо в render-теле компонента (`UpcomingEventBadge`). Антон поймал это в своём checkout. Code-reviewer субагент указывал на hydration-риск того же `Date.now()`, я добавил `useHydrated()` — это убрало mismatch, но НЕ убрало impure-вызов из render, lint всё равно красный.

**Как применять:**
- Frontend gate = `npm run typecheck` + `npm run lint` + `npm run build`, все три, проверять **exit code явно** (`; echo $?`), не «пустой output = успех».
- `Date.now()`, `Math.random()`, `new Date()` без аргумента в render-теле React-компонента = purity-нарушение. Выносить на сервер (серверный компонент / data-loader) или в `useEffect`/lazy `useState(() => ...)`.
- Лучшее решение «время-зависимого UI»: вычислять на сервере и передавать пропом — заодно убирает hydration mismatch и делает компонент presentational.

Связано: [[feedback_verification_discipline.md]] (verified vs believed-done; проверять exit codes, не верить пустому output).
