# New Service Skills — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build 5 service skills + 1 orchestrator that automate the user's "spin up a new web service" workflow via official APIs/CLIs.

**Architecture:** Each skill is a self-contained directory (`SKILL.md` + `references/` + `README.md`) mirroring the existing `humanizer/`/`stop-slop/` style. Skills are token/CLI-driven with a shared credential-resolution convention and preflight checks. An orchestrator skill threads outputs to inputs. No browser automation; no unattended spend.

**Tech Stack:** Markdown skills; bash + `curl` + `jq`; `gh` CLI; `railway` CLI + Railway GraphQL; Cloudflare API v4; reg.ru API v2; Yandex Metrika Management API.

**Note on "tests":** These artifacts are skill documents, not compiled code. "Verification" = frontmatter lint, request-shape validation against docs, and live read-only / dry-run probes where credentials exist (GitHub, Railway). TDD steps are replaced by concrete verification commands per task.

---

## File structure

```
regru-domain/        SKILL.md  references/api.md     README.md
github-repo/         SKILL.md  references/landing.md README.md
cloudflare-setup/    SKILL.md  references/api.md     README.md
railway-deploy/      SKILL.md  references/api.md     README.md
yandex-metrica/      SKILL.md  references/api.md     README.md
new-service/         SKILL.md  references/flow.md    README.md
references/credentials.md      # shared credential convention, linked by all
README.md                      # repo index, add the 6 skills
```

Shared convention (in `references/credentials.md`, referenced by each skill):
resolve creds env → `~/.config/<service>/token` → ask user; always preflight.

---

## Task 1: Shared credential reference + repo README

**Files:** Create `references/credentials.md`; Modify `README.md`.

- [ ] Write `references/credentials.md`: resolution order, env var table, preflight pattern, money/irreversible-action gating rule.
- [ ] Add the 6 skills to repo `README.md` skill list.
- [ ] Verify: `test -f references/credentials.md`; README lists all 6.
- [ ] Commit.

## Task 2: github-repo skill

**Files:** Create `github-repo/SKILL.md`, `github-repo/references/landing.md`, `github-repo/README.md`.

- [ ] SKILL.md frontmatter: name `github-repo`, description (trigger: create repo + landing page), allowed-tools: Bash, Read, Write, Edit, AskUserQuestion.
- [ ] Body: inputs (owner/name, project name, description, brief, visibility); preflight `gh auth status`; scaffold plain meaningful `index.html` (no CSS framework) from project brief; `gh repo create` + push; `--scaffold-only` path; outputs (repo URL, branch, clone path).
- [ ] references/landing.md: HTML landing template + how to fill it from a brief.
- [ ] Verify (LIVE): scaffold-only run builds `index.html` locally without pushing; frontmatter parses.
- [ ] Commit.

## Task 3: railway-deploy skill

**Files:** Create `railway-deploy/SKILL.md`, `railway-deploy/references/api.md`, `railway-deploy/README.md`.

- [ ] SKILL.md frontmatter: name `railway-deploy`, allowed-tools: Bash, Read, AskUserQuestion.
- [ ] Body: inputs (repo URL/dir, project name, optional custom domain); preflight `railway whoami`; create project + deploy from GitHub; read generated domain; attach custom domain → return CNAME target (GraphQL where CLI lacks it); outputs.
- [ ] references/api.md: railway CLI commands + GraphQL queries (customDomainCreate, project/service/domain reads) with exact request bodies.
- [ ] Verify (LIVE read-only): `railway whoami`, `railway list`; GraphQL shapes documented.
- [ ] Commit.

## Task 4: cloudflare-setup skill

**Files:** Create `cloudflare-setup/SKILL.md`, `cloudflare-setup/references/api.md`, `cloudflare-setup/README.md`.

- [ ] SKILL.md frontmatter: name `cloudflare-setup`, allowed-tools: Bash, Read, AskUserQuestion.
- [ ] Body: inputs (domain, DNS records, proxied); preflight `GET /user/tokens/verify`; create zone → read assigned nameservers; apply SSL=full, Always Use HTTPS, Auto HTTPS Rewrites, Security Level=medium, Bot Fight Mode; create DNS records; outputs (zone ID, NS, record IDs).
- [ ] references/api.md: CF API v4 curl templates for each call with exact endpoints/payloads.
- [ ] Verify: request shapes match API v4 docs; preflight present. (No live token → structural.)
- [ ] Commit.

## Task 5: regru-domain skill

**Files:** Create `regru-domain/SKILL.md`, `regru-domain/references/api.md`, `regru-domain/README.md`.

- [ ] SKILL.md frontmatter: name `regru-domain`, allowed-tools: Bash, Read, AskUserQuestion.
- [ ] Body: inputs (domain, target NS, action check|set-ns|buy); preflight `domain/nop`; set-ns via `domain/update_nss`; buy is dry-run default + explicit confirm; outputs.
- [ ] references/api.md: reg.ru API v2 curl templates (nop, check, get_nss, update_nss, create) with auth params.
- [ ] Verify: request shapes match reg.ru API v2 docs; buy gated.
- [ ] Commit.

## Task 6: yandex-metrica skill

**Files:** Create `yandex-metrica/SKILL.md`, `yandex-metrica/references/api.md`, `yandex-metrica/README.md`.

- [ ] SKILL.md frontmatter: name `yandex-metrica`, allowed-tools: Bash, Read, Edit, AskUserQuestion.
- [ ] Body: inputs (domain, repo path/URL, counter name); preflight `GET /management/v1/counters`; create counter; build snippet; inject into `index.html` (idempotent); commit + push → redeploy; outputs (counter ID, snippet, commit SHA).
- [ ] references/api.md: Management API create-counter request + tracking snippet template + OAuth token how-to.
- [ ] Verify: request shape matches Management API docs; snippet injection idempotent.
- [ ] Commit.

## Task 7: new-service orchestrator skill

**Files:** Create `new-service/SKILL.md`, `new-service/references/flow.md`, `new-service/README.md`.

- [ ] SKILL.md frontmatter: name `new-service`, allowed-tools: Skill, Bash, Read, Write, AskUserQuestion.
- [ ] Body: inputs (project name, domain, github path, brief); step sequence invoking the 5 skills, threading outputs→inputs; preflight aggregate (check all creds up front, report missing); `--dry-run` prints planned steps without side effects; spend/irreversible steps gated.
- [ ] references/flow.md: the data-flow diagram + exact input/output contract for each step.
- [ ] Verify: dry-run prints the full ordered plan with threaded values; frontmatter parses.
- [ ] Commit.

## Task 8: Final pass

- [ ] Lint all 6 `SKILL.md` frontmatters parse (name/description/allowed-tools present).
- [ ] Update repo README install section if needed.
- [ ] Write a morning summary note (what's built, what needs the user's tokens, how to run the live pipeline).
- [ ] Commit.

---

## Self-review

- **Spec coverage:** github-repo→Task2, railway→Task3, cloudflare→Task4, regru→Task5, metrica→Task6, orchestrator→Task7, credential convention→Task1. All covered.
- **Consistency:** skill names match spec table; credential env var names match `references/credentials.md`.
- **No placeholders:** each task names exact files + concrete verification command.
