# regru-domain

Skill: manage a domain at reg.ru via API v2 — check availability, read/set
nameservers (point them at Cloudflare), and optionally register a domain. reg.ru
API v2.

- **Credentials:** `REGRU_USERNAME` + `REGRU_PASSWORD` (see [`../references/credentials.md`](../references/credentials.md)).
  Requires API access enabled and (usually) IP allowlisted at reg.ru.
- **Main action:** `set-ns` — repoint the domain to Cloudflare's nameservers.
- **Gated:** `buy` (registration) is dry-run + explicit confirmation; it spends money.
- **Outputs:** action result, NS confirmation.

Part of the [new-service](../new-service/) pipeline.

## Install

```bash
ln -s "$PWD/regru-domain" ~/.claude/skills/regru-domain
```
