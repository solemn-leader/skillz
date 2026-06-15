# Landing page reference

Plain, semantic HTML. No CSS framework, no CDN, no build step. The page should be
banal in form but real in content — a visitor must understand what the project is.

## Template

Fill the `{{...}}` slots from the inputs. Keep the inline `<style>` tiny and
optional (a few readability rules), or drop it entirely — never pull in a framework.

```html
<!DOCTYPE html>
<html lang="{{lang|ru}}">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{project_name}} — {{description}}</title>
  <meta name="description" content="{{description}}">
  <!-- analytics: yandex-metrica counter goes here -->
</head>
<body>
  <header>
    <h1>{{project_name}}</h1>
    <p>{{description}}</p>
  </header>

  <main>
    <section>
      <h2>Что это</h2>
      <p>{{brief_paragraph}}</p>
    </section>

    <section>
      <h2>Как это работает</h2>
      <ul>
        <li>{{point_1}}</li>
        <li>{{point_2}}</li>
        <li>{{point_3}}</li>
      </ul>
    </section>

    <section>
      <h2>Попробовать</h2>
      <p><a href="{{cta_href|#}}">{{cta_label|Связаться}}</a></p>
    </section>
  </main>

  <footer>
    <p>&copy; {{year}} {{project_name}}</p>
  </footer>
</body>
</html>
```

## Optional readability style

If you include a `<style>`, keep it to a handful of rules and inline in `<head>`:

```html
<style>
  body { font-family: system-ui, sans-serif; max-width: 42rem; margin: 2rem auto;
         padding: 0 1rem; line-height: 1.6; }
  h1 { margin-bottom: 0.25rem; }
  header p { color: #555; font-size: 1.1rem; }
  section { margin: 2rem 0; }
</style>
```

## Writing the copy from a brief

- **Hero (`h1` + subtitle):** project name, then the one-line pitch verbatim.
- **"Что это":** 2–4 sentences expanding the brief into plain language — what
  problem it solves and for whom. No marketing fluff, no "revolutionary", no
  rule-of-three filler.
- **"Как это работает":** three concrete bullets. If the brief doesn't give three,
  write fewer real ones rather than padding.
- **CTA:** a single clear action (link to the app, a contact, a waitlist). Default
  label "Связаться" / href `#` if nothing better is known.
- **lang:** default `ru` for Russian projects; switch to `en` if the brief is English.

## The analytics placeholder

Always leave this exact comment right before `</head>`:

```html
<!-- analytics: yandex-metrica counter goes here -->
```

The `yandex-metrica` skill finds this marker and replaces it with the counter
snippet, so the injection stays idempotent.
