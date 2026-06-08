# claude-skills

Скиллы Claude Code + память + конфиг Антона. На новой машине достаточно одного `git clone`, чтобы получить рабочую среду.

---

## Установка на новой машине

```bash
# 1. Скиллы
git clone --recurse-submodules https://github.com/Antthonyyy/claude-skills.git ~/.claude/skills

# 2. Память (симлинком, чтобы Claude видел её по нужному пути)
mkdir -p ~/.claude/projects/-Users-mac-targetolog-ai-assistant
ln -s ~/.claude/skills/_memory ~/.claude/projects/-Users-mac-targetolog-ai-assistant/memory

# 3. Конфиг (settings.json, commands/)
ln -s ~/.claude/skills/_config/settings.json ~/.claude/settings.json
ln -s ~/.claude/skills/_config/commands     ~/.claude/commands

# 4. Агенты — отдельный репо
git clone https://github.com/Antthonyyy/claude-agents.git ~/.claude/agents
```

Если путь к проекту `targetolog-ai-assistant` на новой машине отличается от `/Users/mac/targetolog-ai-assistant`, поправь имя папки во втором шаге — Claude Code хеширует путь рабочего каталога в имя `~/.claude/projects/<slug>/`.

`.env` репо `targetolog-ai-assistant` зашифрован git-crypt — ключ нужно переносить отдельно (не через git):

```bash
# На старой машине:
git-crypt export-key /path/to/usb/targetolog-gc.key
# На новой (после git clone targetolog-ai-assistant):
cd targetolog-ai-assistant
git-crypt unlock /path/to/usb/targetolog-gc.key
```

---

## Структура

```text
skills/
├── <skill>/SKILL.md          # все исполнимые скиллы Claude Code
├── _memory/                  # автопамять (правила, проектный контекст, feedback)
├── _config/
│   ├── settings.json
│   └── commands/
└── notebooklm/               # сабмодуль PleasePrompto/notebooklm-skill
```

Агенты (`~/.claude/agents/`) — отдельный репо `Antthonyyy/claude-agents.git`, не лежат внутри `claude-skills`.

Папки с префиксом `_` Claude Code не парсит как скиллы (там нет `SKILL.md`).

---

## Обновление с другой машины

```bash
cd ~/.claude/skills
git pull
# Если симлинки правильно поставлены — память и конфиг обновятся автоматически.
```

После правки памяти/конфига на любой машине — закоммитить и запушить тот же путь.
