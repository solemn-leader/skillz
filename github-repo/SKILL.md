---
name: github-repo
description: |
  Create a GitHub repository and scaffold a simple, meaningful plain-HTML landing
  page, then push it. Use when the user wants a new repo with a starter landing
  page (often as the deploy source for Railway/Vercel/etc.), or asks to "make me a
  repo with a landing" for a project. Uses the gh CLI. Part of the new-service
  pipeline.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
---

# github-repo: create a repo with a landing page

Create a GitHub repository at a given path and scaffold a simple, **meaningful**
plain-HTML landing page (no CSS framework — banal HTML, but with real content),
then push. Designed to be a clean deploy source for Railway.

## Inputs

Gather these (ask only for what's missing, prefer one `AskUserQuestion`):

- **repo path** — `owner/name` (e.g. `solemn-leader/tenderman`).
- **project name** — display name (e.g. `tendERR`).
- **description** — one line; becomes the repo description and the page subtitle.
- **brief** *(optional)* — a few sentences about what the project does; used to
  write the landing copy. If absent, derive sensible copy from name + description.
- **visibility** — `public` (default) or `private`.
- **source URL** *(optional)* — a reference project the landing is modeled on.

## Step 1 — Preflight

See [`../references/credentials.md`](../references/credentials.md). GitHub uses the
`gh` CLI.

```bash
gh auth status || { echo "Run: gh auth login"; exit 1; }
```

Confirm the authenticated account owns (or can push to) the target `owner`. If the
repo already exists, stop and ask whether to reuse, rename, or abort — never force.

```bash
gh repo view "$REPO_PATH" >/dev/null 2>&1 && echo "EXISTS: $REPO_PATH"
```

## Step 2 — Scaffold the landing locally

Create a clean working dir and write the files. Use the template and copy guidance
in [`references/landing.md`](references/landing.md). The landing MUST:

- be a single `index.html` of plain semantic HTML (no Tailwind/Bootstrap/CDN CSS);
- actually say what the project is — hero with project name + the one-line pitch,
  a short "what it does" section derived from the brief, and a clear call to action;
- include `<title>`, `<meta name="description">`, viewport, and `lang`;
- leave an obvious, commented placeholder where analytics (Yandex Metrica) will be
  injected later by the `yandex-metrica` skill:
  `<!-- analytics: yandex-metrica counter goes here -->` just before `</head>`.

Also write a minimal `README.md` (project name + description + "Deployed on Railway")
and a `.gitignore` (OS/editor junk).

```bash
WORK="$(mktemp -d)/${REPO_PATH##*/}"
mkdir -p "$WORK" && cd "$WORK"
# write index.html, README.md, .gitignore  (see references/landing.md)
git init -q && git add -A && git commit -q -m "Initial landing page"
```

## Step 3 — Create the repo and push

```bash
gh repo create "$REPO_PATH" \
  --"$VISIBILITY" \
  --description "$DESCRIPTION" \
  --source "$WORK" --remote origin --push
```

`gh repo create --source ... --push` creates the remote and pushes the existing
commit in one go. Confirm the default branch name (`main`).

## Step 4 — Output

Report back, and (when called by the orchestrator) return these for the next step:

- `repo_url` — `https://github.com/<owner>/<name>`
- `default_branch` — usually `main`
- `clone_path` — `$WORK` (local checkout, reused by `yandex-metrica`)

## Modes

- **`--scaffold-only`** — do Steps 1–2 only: build the files locally and print the
  path, no remote, no push. Use this to preview the landing or to verify the skill
  without touching GitHub.

## Notes

- Plain HTML by request — do not add a CSS framework or JS bundler.
- Never `--force` push or overwrite an existing repo's history.
- Keep the commit history clean: one initial commit is enough.
