# New Service Skills — Design

**Date:** 2026-06-15
**Author:** Claude (brainstormed with solemn-leader)
**Status:** Approved (overnight autonomous build)

## Goal

A set of reusable skills that let the user spin up a brand-new web service in a
single request. Each external service used in the user's manual workflow gets its
own self-contained skill, plus an orchestrator that chains them in the right order.

User's manual workflow being automated:

1. Buy a domain on reg.ru
2. Make a GitHub repo with a simple landing page (deploy target = Railway)
3. Connect the domain to Cloudflare for HTTPS + basic protection; point reg.ru
   nameservers at Cloudflare
4. Deploy to Railway, add the site link, add the Railway-provided DNS records to
   Cloudflare
5. Connect Yandex Metrica

## Decisions (locked during brainstorming)

- **Execution mechanism:** Official APIs / CLIs with tokens. No browser automation.
  Chosen for reliability during unattended runs.
- **Scope:** 5 service skills + 1 orchestrator.
- **Tonight's deliverable:** Build and verify the skills only. No domain purchase,
  no live production deploy. Live pipeline run happens later with the user present.

## Environment (probed 2026-06-15)

- `gh` 2.88.1 — authenticated as `solemn-leader`. GitHub skill verifiable live.
- `railway` 4.40.0 — logged in as Daniil Zimerman (solemnda@gmail.com). Railway
  skill verifiable live.
- `curl` + `jq` present.
- No Cloudflare / reg.ru / Yandex tokens in env. Those skills are written against
  documented APIs and verified structurally (request shape, preflight checks),
  not against the live service tonight.

## Skill format

Each skill is a directory under the repo root, mirroring `humanizer/` and
`stop-slop/`:

```
<skill-name>/
  SKILL.md          # YAML frontmatter (name, description, allowed-tools) + instructions
  references/       # API details, request templates, field references
  README.md         # human-facing summary + install
```

Symlinked into `~/.claude/skills/<name>` (and `~/.agents/skills/` for Codex).

### Shared credential convention

Every service skill resolves credentials in this order, and performs a
**preflight access check** before doing any real work:

1. Environment variable (named below)
2. `~/.config/<service>/token` (or service-specific file)
3. Ask the user, with a short note on how to obtain the credential

| Skill            | Credential                                  |
|------------------|---------------------------------------------|
| github-repo      | `gh` CLI auth (or `GH_TOKEN`)               |
| railway-deploy   | `railway` CLI login (or `RAILWAY_TOKEN`)    |
| cloudflare-setup | `CLOUDFLARE_API_TOKEN`                       |
| regru-domain     | `REGRU_USERNAME` + `REGRU_PASSWORD`         |
| yandex-metrica   | `YANDEX_METRIKA_TOKEN` (OAuth)              |

Any action that spends money (domain purchase) or is hard to reverse defaults to
**dry-run + explicit confirmation**.

## The skills

### 1. github-repo

- **Purpose:** Create a GitHub repo at a given path and scaffold a simple,
  meaningful plain-HTML landing page (no CSS framework), then push.
- **Inputs:** repo path (`owner/name`), project name, one-line description,
  optional longer brief, visibility.
- **Outputs:** repo URL, default branch, local clone path.
- **Mechanism:** `gh repo create`, write `index.html` + minimal files, `git push`.
- **Verify:** live dry-run via a throwaway repo created then deleted, or
  `--scaffold-only` that builds the files locally without pushing.

### 2. railway-deploy

- **Purpose:** Create a Railway project from a GitHub repo, deploy it, return the
  generated domain; optionally attach a custom domain and return the CNAME target
  that Cloudflare needs.
- **Inputs:** repo URL (or linked dir), project name, optional custom domain.
- **Outputs:** project/service IDs, generated `*.up.railway.app` domain, custom
  domain CNAME target.
- **Mechanism:** `railway` CLI + Railway public GraphQL API for steps the CLI
  doesn't cover (custom domain → CNAME target).
- **Verify:** live read-only calls (`railway whoami`, `railway list`); request
  templates validated against API docs.

### 3. cloudflare-setup

- **Purpose:** Add a domain as a Cloudflare zone, return assigned nameservers,
  enable HTTPS baseline + basic protection, manage DNS records.
- **Inputs:** domain, target DNS records (e.g. CNAME root/www → Railway target),
  proxied flag.
- **Outputs:** zone ID, assigned nameservers, created record IDs.
- **Settings applied:** SSL mode `full`, Always Use HTTPS on, Automatic HTTPS
  Rewrites on, Security Level `medium`, Bot Fight Mode on.
- **Mechanism:** Cloudflare API v4 via `curl` + `jq`.
- **Verify:** request shapes against API v4 docs; preflight `GET /user/tokens/verify`.

### 4. regru-domain

- **Purpose:** Check a domain, optionally purchase it (gated), and set its
  nameservers to Cloudflare's.
- **Inputs:** domain, target nameservers, action (check | set-ns | buy).
- **Outputs:** domain status, NS update result.
- **Mechanism:** reg.ru API v2 (`domain/get_nss`, `domain/update_nss`,
  `domain/check`, `domain/create`) via `curl` + `jq`.
- **Verify:** request shapes against reg.ru API v2 docs; `buy` is dry-run by
  default and requires explicit confirmation.

### 5. yandex-metrica

- **Purpose:** Create a Metrica counter for the domain, get the tracking snippet,
  inject it into the landing page, commit, and trigger a redeploy.
- **Inputs:** domain, repo (local path or URL), counter name.
- **Outputs:** counter ID, snippet, commit SHA.
- **Mechanism:** Yandex Metrika Management API (`POST /management/v1/counters`)
  via `curl`; edit `index.html`; `git commit && push`.
- **Verify:** request shape against Management API docs; preflight token check
  via `GET /management/v1/counters`.

### 6. new-service (orchestrator)

- **Purpose:** Run the whole pipeline from one request.
- **Inputs:** project name, domain, GitHub repo path, short description/brief.
- **Flow:**
  ```
  github-repo            -> repo URL, clone path
  railway-deploy(repo)   -> railway domain + custom-domain CNAME target
  cloudflare-setup(domain, CNAME->target, proxied)
                         -> CF nameservers
  regru-domain(domain, set-ns=CF nameservers)
  yandex-metrica(domain, repo)  -> snippet committed -> Railway redeploys
  -> final summary with every link and ID
  ```
- **Gating:** domain purchase and any spend are excluded from the autonomous path
  and require explicit confirmation. Each step does its own preflight; the
  orchestrator stops and reports clearly if a credential or prior output is missing.
- **Mechanism:** invokes the five skills in order, threading outputs to inputs.

## Testing strategy (tonight)

- **Frontmatter lint:** every `SKILL.md` parses, has `name`/`description`/`allowed-tools`.
- **github-repo:** live scaffold-only run; optionally create+delete a throwaway repo.
- **railway-deploy:** live read-only calls; request templates checked vs docs.
- **cloudflare-setup / regru-domain / yandex-metrica:** structural validation —
  request shapes match documented APIs, preflight checks present, dry-run paths work.
- **new-service:** dry-run that prints the planned step sequence and the
  inputs/outputs it would thread, without executing side effects.

## Out of scope

- Buying domains or any paid action without explicit per-run confirmation.
- Browser-automation fallbacks (token/CLI only, per decision).
- CSS frameworks / fancy landing pages (plain meaningful HTML only).
