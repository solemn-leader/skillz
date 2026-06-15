# cloudflare-setup

Skill: add a domain to Cloudflare (create zone), return the assigned nameservers,
apply an HTTPS + basic-protection baseline, and manage DNS records. Cloudflare API v4.

- **Credentials:** `CLOUDFLARE_API_TOKEN` (see [`../references/credentials.md`](../references/credentials.md)).
- **Baseline applied:** SSL `full`, Always Use HTTPS, Automatic HTTPS Rewrites,
  Security Level `medium`, Bot Fight Mode on.
- **Outputs:** `zone_id`, `nameservers` (for reg.ru), `record_ids`.

Part of the [new-service](../new-service/) pipeline.

## Install

```bash
ln -s "$PWD/cloudflare-setup" ~/.claude/skills/cloudflare-setup
```
