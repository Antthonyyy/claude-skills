---
name: feedback-railway-variables-set-overwrites
description: railway variables --set заменяет переменную целиком (не дописывает) — читать текущее значение ПЕРЕД set
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 67a40385-1ed7-4a26-bee7-a4a080f5da36
---

`railway variables --set "KEY=VALUE" --service X` **перезаписывает** переменную целиком, а не добавляет к существующему значению. Для список-переменных (CSV вроде `WNW_MANAGER_IDS`) это значит: новый set сотрёт всех, кого там уже было.

**Why:** 2026-06-02 при добавлении Галины в `WNW_MANAGER_IDS` на проде `tg-outreach-listener` `--set "...=950531308,@Galina_Poltavskay"` стёр уже стоявшее `652907258,@tatyanaclub` (Татьяну). В `.env` переменной не было → ложно считал whitelist пустым (классика [[feedback_endpoint_exists_vs_used_in_prod]]: прод-env ≠ .env). Поймал проверкой значения сразу после set, восстановил merged по явной авторизации Антона. Safety-классификатор тоже заблокировал set с inferred-значением Татьяны — правильно.

**How to apply:** перед любым `railway variables --set` для list/CSV-переменной — сначала `railway variables --service X | grep KEY`, взять текущее значение, дописать своё, выставить merged. Не доверять `.env` как источнику истины для прод-env. Связано с [[project_wnw_railway_migration_to_client]], гейт аккаунта обязателен.
