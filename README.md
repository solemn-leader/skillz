# skillz

Коллекция навыков для Claude и других LLM.

Поверх них я использую [obra/Superpowers](https://github.com/obra/Superpowers) — фреймворк навыков для Claude Code (brainstorming, TDD, systematic debugging и т.д.).

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

Один навык:

```bash
ln -s "$PWD/stop-slop" ~/.claude/skills/stop-slop
```

Все сразу (запускать из корня репозитория):

```bash
mkdir -p ~/.claude/skills
ln -sfn "$PWD/cloudflare-setup" ~/.claude/skills/cloudflare-setup
ln -sfn "$PWD/github-repo"      ~/.claude/skills/github-repo
ln -sfn "$PWD/humanizer"        ~/.claude/skills/humanizer
ln -sfn "$PWD/new-service"      ~/.claude/skills/new-service
ln -sfn "$PWD/railway-deploy"   ~/.claude/skills/railway-deploy
ln -sfn "$PWD/regru-domain"     ~/.claude/skills/regru-domain
ln -sfn "$PWD/stop-slop"        ~/.claude/skills/stop-slop
ln -sfn "$PWD/yandex-metrica"   ~/.claude/skills/yandex-metrica
```

`-sfn` пересоздаёт ссылку, если она уже есть, и не лезет внутрь существующего симлинка-папки.

## Как добавить навык в Codex

Тот же `SKILL.md`, только папка другая — `~/.agents/skills/` (персонально) или `.agents/skills/` в проекте.

```bash
ln -s "$PWD/stop-slop" ~/.agents/skills/stop-slop
```

Все сразу:

```bash
mkdir -p ~/.agents/skills
ln -sfn "$PWD/cloudflare-setup" ~/.agents/skills/cloudflare-setup
ln -sfn "$PWD/github-repo"      ~/.agents/skills/github-repo
ln -sfn "$PWD/humanizer"        ~/.agents/skills/humanizer
ln -sfn "$PWD/new-service"      ~/.agents/skills/new-service
ln -sfn "$PWD/railway-deploy"   ~/.agents/skills/railway-deploy
ln -sfn "$PWD/regru-domain"     ~/.agents/skills/regru-domain
ln -sfn "$PWD/stop-slop"        ~/.agents/skills/stop-slop
ln -sfn "$PWD/yandex-metrica"   ~/.agents/skills/yandex-metrica
```
