---
name: railway-deploy
description: |
  Deploy a GitHub repository to Railway, return the generated public domain, and
  (optionally) attach a custom domain — returning the DNS records (CNAME target)
  needed for Cloudflare. Use when the user wants to deploy a repo/site to Railway,
  get its live URL, or wire a custom domain to a Railway service. Uses the railway
  CLI. Part of the new-service pipeline.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

# railway-deploy: deploy a repo to Railway

Create a Railway project from a GitHub repo, deploy it, return the generated
`*.up.railway.app` domain, and optionally attach a custom domain plus the DNS
records Cloudflare needs.

## Inputs

- **repo** — `owner/name` of the GitHub repo to deploy (must be connected to the
  Railway account's GitHub app — see Prerequisites).
- **project name** — Railway project/service name (default: repo name).
- **workspace** *(optional)* — Railway workspace ID/name if the account has more
  than one; required for non-interactive `init`.
- **custom domain** *(optional)* — e.g. `tenderman.ru`; if given, the skill returns
  the DNS records to add in Cloudflare.

## Prerequisites

- `railway` CLI logged in (see [`../references/credentials.md`](../references/credentials.md)).
- The GitHub repo must be accessible to Railway's GitHub app. If `railway add
  --repo` fails with a permissions error, the user must connect GitHub at
  <https://railway.com/account> (install the Railway GitHub app on the repo/owner).

## Step 1 — Preflight

```bash
railway whoami || { echo "Run: railway login"; exit 1; }
```

If a workspace is needed and not provided, list and ask:
`railway list --json` shows projects grouped by workspace.

## Step 2 — Create the project

```bash
railway init --name "$PROJECT" ${WORKSPACE:+--workspace "$WORKSPACE"} --json
```

Capture the `projectId` from the JSON. `init` links the new project to the current
directory automatically. (If you are not in a project dir, run from a scratch dir;
the GitHub-repo deploy in Step 3 does not need local source.)

## Step 3 — Add the service from the GitHub repo (triggers deploy)

```bash
railway add --service "$PROJECT" --repo "$REPO" --json
```

This creates a service linked to the GitHub repo and kicks off the first
deployment. Railway auto-builds (Nixpacks/static) — a plain-HTML repo is served as
a static site. Watch status:

```bash
railway status --json
railway logs --service "$PROJECT" 2>&1 | tail -20   # build/deploy logs
```

> Alternative — deploy local source instead of a connected repo:
> from the repo checkout run `railway up --service "$PROJECT" --ci`. Prefer
> `--repo` for the pipeline so Railway redeploys on future pushes (needed by the
> `yandex-metrica` step).

## Step 4 — Generate the public Railway domain

```bash
railway domain --service "$PROJECT" --json
```

No domain argument = Railway generates a `*.up.railway.app` domain (max 1 per
service). Capture the `domain` field — this is the immediate live URL.

## Step 5 — (Optional) Attach the custom domain → get DNS records

```bash
railway domain "$CUSTOM_DOMAIN" --service "$PROJECT" --json
```

Supplying a custom domain returns the **required DNS records** (a CNAME whose
target is something like `<hash>.up.railway.app`, or Railway's value). Capture that
target — the `cloudflare-setup` skill creates a proxied CNAME pointing to it.

If the JSON shape changes, see [`references/api.md`](references/api.md) for the
GraphQL fallback (`customDomainCreate`) that returns the same `cname` target.

## Step 6 — Output

Return for the orchestrator:

- `project_id`, `service_name`
- `railway_domain` — the `*.up.railway.app` URL (works immediately)
- `custom_domain` and `cname_target` — for Cloudflare (if a custom domain was set)

## Notes

- `--repo` deploys keep auto-deploying on push, so later commits (e.g. the Metrica
  snippet) redeploy automatically. With `railway up` you must redeploy manually.
- Don't `railway delete` a project as part of normal runs — that's destructive and
  gated.
