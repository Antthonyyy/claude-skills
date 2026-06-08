---
name: dont-drift-diagnose-to-deploy
description: "При дрейфе сессии из read-only диагностики в правку кода + прод-деплой — пауза, перезапуск через /orchestrate или /critique, обязательная сверка build-context vs git status"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e397c182-b1ca-478a-885f-d18d857ae7dd
---

Когда сессия началась как диагностический скилл (например `/wnw-outreach-check`, `/spec-miner`, `/code-reviewer`) и пользователь по ходу даёт «давай делать» — это **смена режима**, не продолжение того же скилла. Перед тем как писать код и тем более деплоить, **переключиться в `/orchestrate`** (план → автокритика с WebSearch → исполнение по шагам Karpathy → gate) или хотя бы прогнать план через `/critique`.

Конкретный must-do шаг перед `railway up` / любым прод-деплоем, который пакует файлы из рабочего дерева (не образ): **пересечь список путей, что копирует deploy-скрипт, с `git status` по тем же путям**. Если хоть один путь грязный — WIP утечёт в прод-контейнер. Деплой остановить, собрать чистый build-context из `git show HEAD:path` + точечный фикс.

**Why:** 03.06.2026 в сессии `/wnw-outreach-check` после диагностики я начал делать persistent-логи listener'а через `Dockerfile.listener`, прошёл шаги «diff → AskUserQuestion deploy» без плана/критики. Антон поймал глазами, что `scripts/deploy_outreach_railway.sh` копирует `scripts/outreach_core/` и `scripts/outreach_db.py` как директории/файлы из рабочего дерева, а в них висел WIP (модифицированные `outreach_db.py`, `outreach_core/public_sync.py` + новые `payment_*`/`phone_normalize.py`). При запуске скрипта весь этот WIP уехал бы на прод-листенер. Тип ошибки тот же, что в [[ground-recommendations-in-code]] и [[verification-discipline]] — «verified» выдан за «believed».

**How to apply:**
- Триггер: сессия read-only скилла + просьба «давай сделаем X» / «деплой» / «правка прода».
- Действие 1: перед первой правкой кода прямо сказать пользователю «переключаюсь в режим исполнения» и пересобрать план через `/orchestrate` (или, минимум, `/critique` на план).
- Действие 2: перед `railway up` / любого пакующего рабочее дерево деплоя — обязательная сверка путей build-script'а с `git status -- <те же пути>`. Если грязно — собирать чистый build-context из `git show HEAD:path`.
- Действие 3: разделять в отчёте «verified» (build-context собран, проверен по списку файлов) и «believed» (deploy успешен — до фактического подтверждения по логам).

Связано: [[ground-recommendations-in-code]], [[verification-discipline]], [[execute-and-verify]], [[railway-manual-deploy-per-service]].
