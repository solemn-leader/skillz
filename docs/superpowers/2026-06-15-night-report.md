# Ночной отчёт — скиллы для создания нового сервиса

**Дата:** 2026-06-15 (ночь)
**Ветка:** `feature/new-service-skills`

## Что сделано

Шесть скиллов, автоматизирующих твой алгоритм запуска сервиса. Каждый — папка с
`SKILL.md` + `references/` + `README.md`, в стиле существующих `humanizer`/`stop-slop`.

| Скилл              | Делает |
|--------------------|--------|
| `new-service`      | оркестратор — за 1 запрос гоняет всю цепочку |
| `github-repo`      | создаёт репо + простой осмысленный HTML-лендинг, пушит (`gh`) |
| `railway-deploy`   | деплоит репо в Railway, отдаёт домен + CNAME для кастомного домена |
| `cloudflare-setup` | зона в Cloudflare: HTTPS-база + защита + DNS (API v4) |
| `regru-domain`     | переключает NS домена на Cloudflare (покупка — gated) |
| `yandex-metrica`   | счётчик Метрики + идемпотентная вставка в лендинг |

Плюс `references/credentials.md` — общая конвенция доступов (env → файл → спросить,
всегда preflight, траты/необратимое — только с подтверждением).

Спека: `docs/superpowers/specs/2026-06-15-new-service-skills-design.md`
План: `docs/superpowers/plans/2026-06-15-new-service-skills.md`

## Что проверено (ночью, без трат)

- **Фронтматтеры** всех 6 — парсятся, имя = папке, есть Input/Preflight/Output.
- **Внутренние ссылки** между файлами — все резолвятся.
- **github-repo** — лендинг реально собирается локально, валидный HTML, все
  обязательные элементы + маркер для Метрики на месте.
- **railway-deploy** — живые read-only вызовы (`whoami`, `list`) проходят; команды
  свёрены с реальным `railway --help` (деплой из GitHub через `railway add --repo`,
  кастомный домен через `railway domain <domain>` отдаёт DNS-записи).
- **cloudflare-setup** — формат запросов свёрен с CF API v4, envelope подтверждён.
- **regru-domain** — API reg.ru доступен, envelope `result`/`error_code` подтверждён.
- **yandex-metrica** — Management API доступен, вставка счётчика **идемпотентна**
  (двойной прогон → один сниппет).
- **new-service** — агрегатный preflight реально отрабатывает.

## Что нужно от тебя, чтобы прогнать вживую

Сейчас доступны только **gh** (залогинен) и **railway** (залогинен). Для остального
нужны токены (как получить — в `references/credentials.md`):

- `CLOUDFLARE_API_TOKEN` — Zone/DNS/Settings edit + Account Zones edit
- `REGRU_USERNAME` / `REGRU_PASSWORD` — + включить API и вайтлист IP в панели reg.ru
- `YANDEX_METRIKA_TOKEN` — OAuth, scope `metrika:write`
- Railway: подключить GitHub-приложение к репо, чтобы работал `railway add --repo`

## Как поставить и запустить

```bash
cd ~/hustle/skillz
for s in new-service github-repo railway-deploy cloudflare-setup regru-domain yandex-metrica; do
  ln -s "$PWD/$s" ~/.claude/skills/$s
done
```

Потом, когда будут токены, одним запросом:
> «Запусти new-service: проект tendERR, домен tenderman.ru, репо
> solemn-leader/tenderman, делает <краткое описание>»

Сначала прогони с `--dry-run` — покажет весь план без сайд-эффектов. Покупку домена
я в автономный путь не закладывал: куплю/подтвержу отдельно, по явному «да».

## Открытые вопросы на утро

1. Реальное описание tendERR/tenderman для лендинга — у меня сейчас заглушка по
   названию. Дай пару фраз — перегенерю копирайт.
2. Railway: статика отдаётся через Nixpacks; если захочешь явный статик-сервер
   (например `serve`/nginx) — добавлю в `github-repo` мелкий конфиг.
3. Поля контактов для reg.ru `domain/create` я оставил параметризованными — если
   будем покупать домены через скилл, заполню их твоими данными в отдельном
   защищённом конфиге.
