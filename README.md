# skillz

Коллекция навыков для Claude и других LLM.

## Навыки

- [`stop-slop/`](stop-slop/) — удаление признаков ИИ из прозы (русская адаптация [hardikpandya/stop-slop](https://github.com/hardikpandya/stop-slop)).
- [`humanizer/`](humanizer/) — переписывает англоязычный текст, убирая 33 паттерна ИИ-письма (зеркало [blader/humanizer](https://github.com/blader/humanizer)).

### Создание нового сервиса

Набор скиллов, автоматизирующих весь путь запуска веб-сервиса через официальные API/CLI. Общая конвенция доступов — [`references/credentials.md`](references/credentials.md).

- [`new-service/`](new-service/) — **оркестратор**: за один запрос гоняет всю цепочку ниже.
- [`github-repo/`](github-repo/) — создаёт GitHub-репозиторий с простым лендингом и пушит.
- [`railway-deploy/`](railway-deploy/) — деплоит репозиторий в Railway, отдаёт домен и CNAME для кастомного домена.
- [`cloudflare-setup/`](cloudflare-setup/) — добавляет зону в Cloudflare (HTTPS + базовая защита), управляет DNS.
- [`regru-domain/`](regru-domain/) — переключает NS домена на reg.ru на нейм-серверы Cloudflare (покупка — с подтверждением).
- [`yandex-metrica/`](yandex-metrica/) — создаёт счётчик Яндекс.Метрики и вшивает его в лендинг.

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
