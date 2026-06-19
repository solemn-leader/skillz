# yandex-metrica

Skill: create a Yandex Metrica counter for a site and inject its tracking snippet
into the landing page, then commit so the deploy goes live with analytics. Uses
the Metrika Management API.

- **Credentials:** `YANDEX_METRIKA_TOKEN` (OAuth, scope `metrika:write`; see
  [`../references/credentials.md`](../references/credentials.md)).
- **Idempotent injection:** replaces the `github-repo` placeholder marker; never
  adds a second snippet on re-runs.
- **Outputs:** `counter_id`, `snippet`, `commit_sha`.

Part of the [new-service](../new-service/) pipeline.

## Install

```bash
ln -s "$PWD/yandex-metrica" ~/.claude/skills/yandex-metrica
```
