# railway-deploy

Skill: deploy a GitHub repo to Railway, get the live `*.up.railway.app` domain,
and optionally attach a custom domain (returning the CNAME target for Cloudflare).
Uses the `railway` CLI (GraphQL fallback documented).

- **Credentials:** `railway` CLI login / `RAILWAY_TOKEN` (see [`../references/credentials.md`](../references/credentials.md)).
- **Prerequisite:** the GitHub repo must be connected to Railway's GitHub app for
  `railway add --repo` to work.
- **Outputs:** `project_id`, `service_name`, `railway_domain`, `custom_domain`,
  `cname_target`.

Part of the [new-service](../new-service/) pipeline.

## Install

```bash
ln -s "$PWD/railway-deploy" ~/.claude/skills/railway-deploy
```
