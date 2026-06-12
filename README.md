# skillz

Коллекция навыков для Claude и других LLM.

## Навыки

- [`stop-slop/`](stop-slop/) — удаление признаков ИИ из прозы (русская адаптация [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop)).
- [`humanizer/`](humanizer/) — переписывает англоязычный текст, убирая 33 паттерна ИИ-письма (зеркало [blader/humanizer](https://github.com/blader/humanizer)).

## Как добавить навык в Claude Code

Слинкуй папку навыка в `~/.claude/skills/` (персонально) или `.claude/skills/` (в проекте) и перезапусти Claude Code — подхватится сам. Симлинк удобнее копии: правки в репо сразу подтягиваются.

```bash
ln -s "$PWD/stop-slop" ~/.claude/skills/stop-slop
```

## Как добавить навык в Codex

Тот же `SKILL.md`, только папка другая — `~/.agents/skills/` (персонально) или `.agents/skills/` в проекте.

```bash
ln -s "$PWD/stop-slop" ~/.agents/skills/stop-slop
```
