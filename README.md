# skillz

Коллекция навыков для Claude и других LLM.

## Навыки

- [`stop-slop/`](stop-slop/) — удаление признаков ИИ из прозы (русская адаптация [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop)).

## Как добавить навык в Claude Code

Скопируй папку навыка в `~/.claude/skills/` (персонально) или `.claude/skills/` (в проекте) и перезапусти Claude Code — подхватится сам.

```bash
cp -r stop-slop ~/.claude/skills/stop-slop
```
