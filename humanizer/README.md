# Humanizer

A skill for Claude Code and OpenCode that removes signs of AI-generated writing from text, making it sound more natural and human.

Зеркало [blader/humanizer](https://github.com/blader/humanizer). Навык работает с англоязычными текстами — детектирует и переписывает 33 паттерна ИИ-письма по гайду Wikipedia «Signs of AI writing».

## Установка

**Claude Code** — склонируй репозиторий и слинкуй папку:

```bash
ln -s "$PWD/humanizer" ~/.claude/skills/humanizer
```

**Codex** — та же папка навыка, другой каталог:

```bash
ln -s "$PWD/humanizer" ~/.agents/skills/humanizer
```

**OpenCode** использует `~/.config/opencode/skills`, но также читает `~/.claude/skills/` для совместимости.

## Использование

Вызови `/humanizer` и передай текст. Опционально можно дать образец своего письма для калибровки голоса — навык подстроится под твой стиль, а не выдаст обезличенный результат:

```
/humanizer Humanize this. Here's a sample of my writing for voice matching: [sample]
```

## Что детектирует

33 пронумерованных паттерна по четырём категориям:

- **Content** — раздувание значимости, упор на медийность, поверхностные `-ing`-обороты, рекламный тон, размытые атрибуции, шаблонные «Challenges» секции.
- **Language** — лексика ИИ, уход от `is/are`, негативные параллелизмы, правило трёх, синонимическая чехарда, ложные диапазоны, пассив.
- **Style** — длинные тире, болд, инлайн-заголовки в списках, Title Case, эмодзи, кудрявые кавычки.
- **Communication / Filler** — артефакты чат-бота, дисклеймеры про cutoff, угодливый тон, слова-заполнители, хеджирование, штампованные позитивные концовки, фальшивые афоризмы.

Навык делает черновик, затем проход-аудит «What makes this obviously AI generated?» и финальную версию без тире.

## Лицензия

MIT. Паттерны основаны на [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing).
