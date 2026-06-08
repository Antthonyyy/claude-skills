---
name: project_wnw_crm_perf_messages_underfetch
description: WNW CRM — перф-боттлнек (full-load-then-filter-in-memory) + латентный баг недобора messages из-за cap PostgREST 1000
metadata: 
  node_type: memory
  type: project
  originSessionId: 447fba0b-f8a1-430c-bde0-dbb388e929d7
---

WNW CRM (`/Users/mac/Downloads/wnw crm`, Next.js 16 + Supabase, отдельный прод Татьяны) — два самых частых экрана (дашборд, список контактов) на каждый запрос грузят всю базу контактов (~913) и обогащают её 7 кросс-табличными батч-запросами через `getContacts()`/`enrichContactsLeadPath()` [src/lib/mockData.ts:437,166], затем фильтруют/считают сегменты/фасеты в памяти Node. Кэша нет (только `force-dynamic`). Дашборд [page.tsx:55] грузит всё ради 4 счётчиков + повторно через `getOutreachFunnelSummary()`. Paged API [route.ts:156] зовёт полный `getContacts()` до пагинации; клиент рефетчит на каждый тоггл фильтра.

**Латентный баг (verified замером 2026-06-04):** PostgREST режет ответ до 1000 строк/запрос — `.limit(5000/10000)` в коде это no-op. `messages` уже недобираются (5855 из 6622 для outreach-когорты) → `messagesCount`/`lastMessageAt`/`lastIncomingAt`/`hasUnreadReply` местами неверны. **Глобальный рост размера чанка (`SUPABASE_IN_BATCH_SIZE=100`) строго ухудшает баг** — только `messages` уязвимы, остальные таблицы ≤100 строк/батч. Фикс — SQL-агрегат/RPC для messages, не «поднять limit».

Индексы базовых таблиц НА МЕСТЕ (wnw-api/schema.sql:143,174,193; schema_external.sql:21,47) — боттлнек в числе RTT + объёме выгрузки, не в индексах. Не переносить lead-path/атрибуцию в materialized view на текущих объёмах (риск дрейфа цифр, инвариант синхронизации в комментариях кода).

**Why:** Антон просил полный анализ перформанса CRM 2026-06-04; план фиксов есть, но код CRM-репо НЕ менялся (отдельный прод, внедрять отдельной задачей с гейтом ветки).
**How to apply:** порядок фиксов P0 = dashboard COUNT-агрегаты → `unstable_cache`+`revalidateTag` в мутациях → messages RPC → bounded parallelism для low-cardinality. Полный отчёт: `clients/wnw/tasks/crm_performance_analysis_2026-06-04.md`. Связано: [[feedback_endpoint_exists_vs_used_in_prod]], [[feedback_wnw_crm_vercel_is_production]].
