# skillz

Коллекция навыков для Claude и других LLM.

## Навыки

- [`stop-slop/`](stop-slop/) — удаление признаков ИИ из прозы (русская адаптация [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop)).

## Как добавить навык в Claude Code

Claude Code ищет навыки в папках `skills/`: персональные — в `~/.claude/skills/`, проектные — в `.claude/skills/` внутри репозитория. Каждый навык — это папка с файлом `SKILL.md`.

### Персональный навык (доступен во всех проектах)

```bash
git clone https://github.com/solemn-leader/skillz.git
mkdir -p ~/.claude/skills
cp -r skillz/stop-slop ~/.claude/skills/stop-slop
```

Должно получиться `~/.claude/skills/stop-slop/SKILL.md`.

### Проектный навык (только для одного репозитория)

```bash
mkdir -p .claude/skills
cp -r /путь/к/skillz/stop-slop .claude/skills/stop-slop
```

### Проверка

Перезапусти Claude Code и набери `/`, либо спроси «какие навыки доступны». Навык `stop-slop` подхватится автоматически по `name` и `description` из фронтматтера `SKILL.md` и активируется, когда ты пишешь или редактируешь текст.
