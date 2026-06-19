---
name: regru-domain
description: |
  Manage a domain at the reg.ru registrar via its API: check availability, read
  nameservers, and — the main job in the new-service flow — point the domain's
  nameservers at Cloudflare. Can also register (buy) a domain, but purchase is
  dry-run by default and requires explicit confirmation because it spends money.
  Use when the user needs to set/repoint reg.ru nameservers (e.g. to Cloudflare),
  check a domain, or register one. Part of the new-service pipeline.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

# regru-domain: registrar operations at reg.ru

Talk to the reg.ru API v2 to check a domain, read/set its nameservers, and
(gated) register it. The core pipeline use is **set the domain's NS to
Cloudflare's** so Cloudflare can manage DNS and HTTPS.

All calls: `POST https://api.reg.ru/api/regru2/<category>/<method>` with form
params `username`, `password`, `input_format=json`,
`output_content_type=json`, and `input_data=<json>`. Details + exact bodies:
[`references/api.md`](references/api.md).

## Inputs

- **domain** — e.g. `tenderman.ru`.
- **action** — `check` | `set-ns` | `buy`.
- **nameservers** *(for set-ns)* — the Cloudflare NS array from `cloudflare-setup`.

## Step 1 — Preflight

See [`../references/credentials.md`](../references/credentials.md). Resolve
`REGRU_USERNAME` / `REGRU_PASSWORD`, then test auth with `nop`:

```bash
curl -s "https://api.reg.ru/api/regru2/domain/nop" \
  --data-urlencode "username=$REGRU_USERNAME" \
  --data-urlencode "password=$REGRU_PASSWORD" \
  --data-urlencode 'input_format=json' \
  --data-urlencode 'output_content_type=json' \
  --data-urlencode 'input_data={"domains":[{"dname":"'"$DOMAIN"'"}]}' \
  | jq '.result'
```

Expect `result: "success"`. reg.ru requires API access enabled and often the
caller IP whitelisted in the control panel — if you get `ACCESS_DENIED_FROM_IP`,
tell the user to whitelist the IP.

## Step 2a — action `check`

```bash
POST domain/check   input_data={"domains":[{"dname":"$DOMAIN"}]}
# -> per-domain availability (avail / unavail / error)
```

## Step 2b — action `set-ns` (the main one)

Point the domain at Cloudflare's nameservers:

```bash
curl -s "https://api.reg.ru/api/regru2/domain/update_nss" \
  --data-urlencode "username=$REGRU_USERNAME" \
  --data-urlencode "password=$REGRU_PASSWORD" \
  --data-urlencode 'input_format=json' \
  --data-urlencode 'output_content_type=json' \
  --data-urlencode 'input_data={"domains":[{"dname":"'"$DOMAIN"'"}],"nss":{"ns0":"'"$NS0"'","ns1":"'"$NS1"'"}}' \
  | jq '.'
```

`ns0`/`ns1` are the two Cloudflare nameservers (Cloudflare assigns exactly two on
the free plan). Verify afterwards with `domain/get_nss`. NS propagation can take
minutes to hours; Cloudflare activates the zone once it sees its NS.

## Step 2c — action `buy` (GATED — spends money)

Registration costs money. **Default to dry-run:** show the price/params and stop.
Only proceed after an explicit "yes" from the user (see
[`../references/credentials.md`](../references/credentials.md) money rule). Then:

```bash
POST domain/create   input_data={"domains":[{"dname":"$DOMAIN"}], ...contacts..., "nss":{...}}
```

Registration requires contact data and (usually) sufficient balance on the reg.ru
account. Gather/confirm contacts with the user before calling. See
[`references/api.md`](references/api.md) for the full `create` payload.

## Step 3 — Output

- `domain`, `action`
- for `set-ns`: the NS set and the `get_nss` confirmation
- for `check`: availability
- for `buy`: order result (only if confirmed)

## Notes

- Never call `domain/create` (or any paid method) on the autonomous path.
- Cloudflare assigns its nameservers when the zone is created — run
  `cloudflare-setup` first, then feed its `nameservers` here.
