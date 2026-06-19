---
name: cloudflare-setup
description: |
  Add a domain to Cloudflare (create a zone), return the assigned nameservers,
  enable an HTTPS + basic-protection baseline, and manage DNS records. Use when
  the user wants to put a domain behind Cloudflare for HTTPS/CDN/protection, get
  the Cloudflare nameservers to set at their registrar, or add/update DNS records
  (e.g. point a domain at Railway). Uses the Cloudflare API v4. Part of the
  new-service pipeline.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

# cloudflare-setup: zone + HTTPS + DNS

Add a domain as a Cloudflare zone, return the nameservers to set at the registrar,
apply an HTTPS and basic-protection baseline, and create DNS records.

All calls use `curl` against `https://api.cloudflare.com/client/v4` with header
`Authorization: Bearer $CLOUDFLARE_API_TOKEN`. Exact payloads:
[`references/api.md`](references/api.md).

## Inputs

- **domain** — apex domain, e.g. `tenderman.ru`.
- **dns records** — list to create, typically one CNAME `@`/`www` → Railway target.
- **proxied** — whether records go through Cloudflare's proxy (default `true`, which
  gives the orange-cloud HTTPS/CDN/protection). Use `false` for DNS-only records.

## Step 1 — Preflight

See [`../references/credentials.md`](../references/credentials.md). Resolve
`CLOUDFLARE_API_TOKEN`, then verify:

```bash
curl -s https://api.cloudflare.com/client/v4/user/tokens/verify \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" | jq '.success, .result.status'
```

Expect `success: true` and `status: "active"`. If not, stop and report.

## Step 2 — Create the zone

Get the account ID, then create the zone:

```bash
ACCOUNT_ID=$(curl -s https://api.cloudflare.com/client/v4/accounts \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" | jq -r '.result[0].id')

curl -s -X POST https://api.cloudflare.com/client/v4/zones \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"'"$DOMAIN"'","account":{"id":"'"$ACCOUNT_ID"'"},"type":"full"}'
```

If the zone already exists, fetch it instead of erroring:
`GET /zones?name=$DOMAIN`. Capture `result.id` (zone ID) and
`result.name_servers` — **these nameservers are the key output**: the user sets
them at reg.ru (the `regru-domain` skill does this).

## Step 3 — HTTPS + basic-protection baseline

Apply these zone settings (each is `PATCH /zones/$ZONE/settings/<key>` with
`{"value":...}`):

| Setting key                | Value     | Why |
|----------------------------|-----------|-----|
| `ssl`                      | `full`    | encrypt edge↔origin; Railway serves HTTPS |
| `always_use_https`         | `on`      | redirect http→https |
| `automatic_https_rewrites` | `on`      | upgrade mixed-content links |
| `security_level`           | `medium`  | baseline challenge level |

Then enable free Bot Fight Mode:

```bash
curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE/bot_management" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" -d '{"fight_mode":true}'
```

See [`references/api.md`](references/api.md) for the exact loop. Report any setting
the token lacks permission to change rather than failing the whole step.

## Step 4 — DNS records

For each requested record (apex CNAME works via Cloudflare's CNAME flattening):

```bash
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE/dns_records" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"CNAME","name":"@","content":"'"$TARGET"'","proxied":true,"ttl":1}'
```

For the new-service flow, create `@` → Railway CNAME target (proxied) and a `www`
CNAME → apex. If a record with the same name/type exists, update it
(`PUT /dns_records/$id`) instead of creating a duplicate.

## Step 5 — Output

- `zone_id`
- `nameservers` — array, hand to `regru-domain`
- `record_ids` — created/updated DNS record IDs

## Notes

- New zones report `status: pending` until the registrar's NS point to Cloudflare;
  that's expected — the `regru-domain` step flips the NS, then Cloudflare activates.
- Deleting a zone is destructive and gated — never part of the autonomous path.
