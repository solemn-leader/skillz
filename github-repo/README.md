# github-repo

Skill: create a GitHub repository and scaffold a simple, meaningful plain-HTML
landing page, then push it. Uses the `gh` CLI.

- **What it does:** gathers project inputs → scaffolds `index.html` (no CSS
  framework) + `README.md` + `.gitignore` → `gh repo create --source --push`.
- **Credentials:** `gh` CLI auth (see [`../references/credentials.md`](../references/credentials.md)).
- **Modes:** `--scaffold-only` builds the files locally without creating a remote.
- **Outputs:** `repo_url`, `default_branch`, `clone_path`.

Part of the [new-service](../new-service/) pipeline.

## Install

```bash
ln -s "$PWD/github-repo" ~/.claude/skills/github-repo
```
