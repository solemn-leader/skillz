---
name: new-service
description: |
  Orchestrate spinning up a brand-new web service end to end in one request:
  create a GitHub repo with a landing page, deploy it to Railway, put the domain
  behind Cloudflare (HTTPS + basic protection), repoint the reg.ru nameservers,
  and add Yandex Metrica. Use when the user wants to launch/bootstrap a new site or
  service from scratch and wire up repo + hosting + domain + DNS + analytics
  together. Chains the github-repo, railway-deploy, cloudflare-setup, regru-domain,
  and yandex-metrica skills.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - Skill
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

# new-service: launch a service end to end

Run the whole pipeline from one request by invoking the five service skills in
order and threading each step's outputs into the next. The exact input/output
contract per step: [`references/flow.md`](references/flow.md).

## Inputs (gather first, one AskUserQuestion)

- **project name** — display name (e.g. `tendERR`).
- **domain** — apex domain you already own or will buy (e.g. `tenderman.ru`).
- **github repo path** — `owner/name` (e.g. `solemn-leader/tenderman`).
- **brief** — a few sentences on what the service does (drives the landing copy).
- **source URL** *(optional)* — a reference project to model the landing on.

## Step 0 — Aggregate preflight

Before doing anything, check every credential up front and report what's missing,
so the run doesn't die halfway. See
[`../references/credentials.md`](../references/credentials.md).

```bash
gh auth status        >/dev/null 2>&1 && echo "github: OK"     || echo "github: MISSING"
railway whoami        >/dev/null 2>&1 && echo "railway: OK"     || echo "railway: MISSING"
[ -n "$CLOUDFLARE_API_TOKEN" ] && echo "cloudflare: token present" || echo "cloudflare: MISSING token"
[ -n "$REGRU_USERNAME" ] && [ -n "$REGRU_PASSWORD" ] && echo "regru: OK" || echo "regru: MISSING"
[ -n "$YANDEX_METRIKA_TOKEN" ] && echo "yandex: token present" || echo "yandex: MISSING token"
```

If anything is missing, list it and ask the user whether to (a) supply it now,
(b) skip that step, or (c) abort. Don't silently skip.

## Step sequence

Invoke each sub-skill with the `Skill` tool, passing the inputs shown and capturing
the outputs into variables threaded forward.

1. **github-repo** — create repo + landing.
   In: repo path, project name, brief, description.
   Out: `repo_url`, `clone_path`.

2. **railway-deploy** — deploy from the repo, attach the custom domain.
   In: `repo` = repo path, project name, `custom_domain` = domain.
   Out: `railway_domain` (immediate live URL), `cname_target` (for Cloudflare).

3. **cloudflare-setup** — zone + HTTPS baseline + DNS.
   In: domain, DNS record `@`/`www` CNAME → `cname_target`, proxied=true.
   Out: `nameservers`, `zone_id`.

4. **regru-domain** — point the registrar at Cloudflare.
   In: domain, action `set-ns`, nameservers = CF `nameservers`.
   Out: NS confirmation.
   (Domain **purchase** is NOT part of this autonomous run — see Gating.)

5. **yandex-metrica** — counter + snippet into the landing.
   In: domain, repo = `clone_path`, counter name = project name.
   Out: `counter_id`, `commit_sha`. The commit/push triggers a Railway redeploy.

## Step final — Summary

Print one block with every artifact:

- repo: `repo_url`
- live now: `https://<railway_domain>`
- target site: `https://<domain>` (active once NS propagate + Cloudflare verifies)
- Cloudflare zone: `zone_id`, nameservers set at reg.ru
- Yandex Metrica counter: `counter_id`
- any steps skipped and why

## Gating

- **Domain purchase and any paid action are excluded** from the autonomous path.
  If the domain isn't registered yet, stop before `set-ns` and tell the user to buy
  it (the `regru-domain` skill's gated `buy` action, with explicit confirmation),
  then resume.
- Each sub-skill runs its own preflight; if one fails mid-run, stop and report
  which step failed and what's needed — don't continue with stale/missing values.

## Modes

- **`--dry-run`** — run Step 0 preflight, then print the full ordered plan with the
  inputs/outputs it would thread between steps, and stop. No repo, deploy, DNS, or
  counter is created. Use this to preview a run.

## Notes

- Order matters: Cloudflare must create the zone (→ nameservers) before reg.ru can
  point at it; Railway must return the `cname_target` before Cloudflare's DNS record
  can be created. The sequence above respects these dependencies.
- See [`references/flow.md`](references/flow.md) for the dependency diagram.
