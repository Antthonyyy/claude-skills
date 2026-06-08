---
name: git-crypt-worktree-workaround
description: "При создании linked git worktree в репо с git-crypt smudge filter падает из-за того, что ключ не копируется в новый worktree. Workaround для случая, когда зашифрованные файлы не нужны."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 11efd386-fde0-4bd0-ade9-cd2237fc749a
---

В репо `targetolog-ai-assistant` ключ git-crypt живёт в `.git/git-crypt/keys/default`. Linked worktree получает собственный `<main>/.git/worktrees/<name>/` без копии ключа → smudge filter падает на `.env` (и любых других `git-crypt` файлах), `git worktree add` завершается с `fatal: ... smudge filter git-crypt failed`.

**Workaround (если зашифрованные файлы НЕ нужны для текущей задачи):**

```bash
git -c filter.git-crypt.smudge=cat -c filter.git-crypt.required=false \
  worktree add <path> -b <branch> <base-sha>

git -C <path> update-index --skip-worktree .env
```

`smudge=cat` пишет blob как есть (зашифрованный), `required=false` снимает строгость, `skip-worktree` убирает шум в `git status`. После этого worktree чистый.

**Если зашифрованные файлы НУЖНЫ:**
- скопировать ключ: `mkdir -p $(git rev-parse --git-dir)/git-crypt/keys && cp <main>/.git/git-crypt/keys/default $(git rev-parse --git-dir)/git-crypt/keys/`
- ИЛИ `git-crypt unlock <keyfile>` в worktree

**Подводный камень при ретрае:** если первая попытка `git worktree add -b <name> <sha>` упала на smudge, **ветка успевает создаться** (worktree-каталог откатывается, ветка — нет). При ретрае с тем же `-b` будет `fatal: a branch named '<name>' already exists`. Решение: ретрай без `-b`, просто `git worktree add <path> <branch>` — переиспользуем уже созданную ветку (она указывает на нужный SHA).

**Why:** git-crypt и git worktrees — known community issue, описано в issue tracker git-crypt. В local memory `[[git-commit-hygiene]]` про git-crypt сказано только «не untrack'ать `.env`», но edge case worktree не покрыт.

**How to apply:** перед любым `git worktree add` в этом репо — сразу использовать форму с `-c filter.git-crypt.smudge=cat -c filter.git-crypt.required=false`, если задача не требует доступа к зашифрованным файлам (auth-кода, секретов, env). Если требует — сначала готовить ключ.

---

**Подводный камень при деплое из такого worktree (`railway up`, `docker build`, любой архиватор каталога):** инструменты, которые архивируют физические файлы каталога, игнорируют `skip-worktree`-флаг git и **отправят зашифрованный blob `.env` на прод**. На Railway `python-dotenv.load_dotenv()` попытается распарсить blob и может тихо проглотить мусор или, хуже, перетереть некоторые ENV псевдослучайными значениями.

**Как защититься:**
- `.env` и `.env.*` должны быть в `.railwayignore` И `.dockerignore` репо.
- **Не перезаписывать** существующие `.railwayignore` через `echo > .railwayignore` — там обычно есть много других нужных исключений (`venv/`, `.git/`, `__pycache__/`, кастомные `projects/`, и т.д.). Проверять `cat` перед изменениями, при необходимости `>>` для дополнения.
- Перед `railway up` из worktree подтвердить, что нужные ENV заданы в Railway service dashboard / CLI — listener полностью полагается на платформенные ENV, файла `.env` в архиве не будет.

В текущем репо `targetolog-ai-assistant` оба ignore-файла уже содержат `.env` и `.env.*` (проверено 2026-05-31). Worktree-деплой безопасен по этой части.

---

Связано: [[git-commit-hygiene]] (общая дисциплина git в этом репо).
