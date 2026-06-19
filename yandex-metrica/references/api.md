# Yandex Metrika reference

## Management API

Base: `https://api-metrika.yandex.net/management/v1`
Auth header: `Authorization: OAuth $YANDEX_METRIKA_TOKEN`
Token: OAuth token with scope `metrika:write`. Get one by registering an app at
<https://oauth.yandex.com/client/new> (Metrika scope) and running the OAuth flow.

### List counters (preflight)
```
GET /counters
-> .counters[] (each has id, name, site)
```

### Create counter
```
POST /counters
{"counter":{"name":"$NAME","site":"$DOMAIN"}}
-> .counter.id
```

Optional counter fields: `code_options`, `goals`, `filters`, `webvisor` — not
needed for a basic counter. Docs:
<https://yandex.com/dev/metrika/en/management/openapi/counters/addcounter>.

## Tracking snippet template

Substitute `COUNTER_ID`. This is the current `tag.js` snippet:

```html
<!-- Yandex.Metrika counter -->
<script type="text/javascript">
   (function(m,e,t,r,i,k,a){m[i]=m[i]||function(){(m[i].a=m[i].a||[]).push(arguments)};
   m[i].l=1*new Date();
   for (var j = 0; j < document.scripts.length; j++) {if (document.scripts[j].src === r) { return; }}
   k=e.createElement(t),a=e.getElementsByTagName(t)[0],k.async=1,k.src=r,a.parentNode.insertBefore(k,a)})
   (window, document, "script", "https://mc.yandex.ru/metrika/tag.js", "ym");

   ym(COUNTER_ID, "init", {
        clickmap:true,
        trackLinks:true,
        accurateTrackBounce:true,
        webvisor:true
   });
</script>
<noscript><div><img src="https://mc.yandex.ru/watch/COUNTER_ID" style="position:absolute; left:-9999px;" alt="" /></div></noscript>
<!-- /Yandex.Metrika counter -->
```

## Injection rule (idempotent)

1. If the marker `<!-- analytics: yandex-metrica counter goes here -->` exists →
   replace it with the snippet.
2. Else if a snippet containing `mc.yandex.ru/metrika/tag.js` exists → replace the
   existing `ym(<old-id>, "init"` and the `watch/<old-id>` with the new id.
3. Else → insert the snippet immediately before `</head>`.

This guarantees exactly one counter snippet per page across re-runs.
