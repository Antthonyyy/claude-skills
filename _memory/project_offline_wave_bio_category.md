---
name: offline-wave bio_category — технический долг
description: build_target_list() не подтягивает bio_category, generic-хук используется всегда
type: project
originSessionId: 3f82212c-014c-4818-9e42-6512939dc827
---
`send_offline_wave.build_target_list()` не достаёт `bio_category` из БД, поэтому `_render_msg()` всегда использует generic offline-хук. Поведенческого бага нет, generic-хук безопасен.

**Why:** низкий приоритет, сознательно оставлено.

**How to apply:** когда понадобится персонализация — добавить `bio_category` в SELECT на [send_offline_wave.py:172](scripts/send_offline_wave.py#L172) и прокинуть в `targets.append(...)`. Существующий `_render_msg()` начнёт использовать профильные хуки без дополнительной логики.
