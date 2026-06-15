---
name: yandex-metrica
description: |
  Create a Yandex Metrica counter for a site and embed its tracking snippet into
  the landing page, then commit so the deploy picks it up. Use when the user wants
  to add Yandex Metrica / Яндекс.Метрика analytics to a site, create a counter, or
  wire the tracking tag into an existing repo. Uses the Metrika Management API.
  Part of the new-service pipeline.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - Bash
  - Read
  - Edit
  - AskUserQuestion
---

# yandex-metrica: counter + snippet injection

Create a Metrica counter via the Management API, then inject its tracking snippet
into the landing page and commit (so Railway redeploys with analytics live).

API calls: `https://api-metrika.yandex.net/management/v1/...` with header
`Authorization: OAuth $YANDEX_METRIKA_TOKEN`. Snippet template + exact request:
[`references/api.md`](references/api.md).

## Inputs

- **domain** — the site, e.g. `tenderman.ru` (the counter's `site`).
- **repo** — local checkout path (preferred, reuses `github-repo`'s `clone_path`)
  or a GitHub URL to clone.
- **counter name** — default: the project/domain name.

## Step 1 — Preflight

See [`../references/credentials.md`](../references/credentials.md). Resolve
`YANDEX_METRIKA_TOKEN` (OAuth, scope `metrika:write`), then verify:

```bash
curl -s "https://api-metrika.yandex.net/management/v1/counters" \
  -H "Authorization: OAuth $YANDEX_METRIKA_TOKEN" | jq '.counters | length'
```

A number (incl. 0) = token works. A `403/401` JSON error = bad/insufficient token;
stop and report.

## Step 2 — Create the counter

```bash
curl -s -X POST "https://api-metrika.yandex.net/management/v1/counters" \
  -H "Authorization: OAuth $YANDEX_METRIKA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"counter":{"name":"'"$NAME"'","site":"'"$DOMAIN"'"}}' \
  | jq '.counter.id'
```

Capture `counter.id`. If a counter for this site already exists and you want to
reuse it, find it in the Step-1 list instead of creating a duplicate.

## Step 3 — Inject the snippet into the landing page

Build the tracking snippet from the template in
[`references/api.md`](references/api.md), substituting `COUNTER_ID`. Then replace
the placeholder marker left by `github-repo`:

```
<!-- analytics: yandex-metrica counter goes here -->
```

Use `Edit` (or `sed`) to swap that exact comment for the snippet. **Idempotent:**
if the marker is gone and a `mc.yandex.ru/metrika/tag.js` snippet is already
present, update the counter ID rather than adding a second snippet. If neither the
marker nor an existing snippet is found, insert the snippet just before `</head>`.

## Step 4 — Commit and redeploy

```bash
cd "$REPO_PATH"
git add index.html
git commit -m "Add Yandex Metrica counter $COUNTER_ID"
git push
```

If Railway is connected to the repo (`railway add --repo`), the push auto-redeploys.
Otherwise run the `railway-deploy` redeploy step (`railway redeploy`).

## Step 5 — Output

- `counter_id`
- `snippet` (what was injected)
- `commit_sha`

## Notes

- Token scope must include write (`metrika:write`); read-only tokens can't create
  counters.
- Keep exactly one counter snippet per page — the idempotent injection guarantees
  this on re-runs.
