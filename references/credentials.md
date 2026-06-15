# Shared credential convention

Every service skill in this repo resolves credentials the same way and performs a
**preflight access check** before doing any real work. This file is the single
source of truth; each skill links here instead of duplicating the rules.

## Resolution order

For each service, resolve the credential in this order and stop at the first hit:

1. **Environment variable** (names below).
2. **Config file** `~/.config/<service>/token` (one credential per line, or the
   service-specific file noted below).
3. **Ask the user** with `AskUserQuestion`, including a one-line note on how to
   obtain the credential (links below). Never invent or guess a token.

After resolving, **always run the preflight check** for that service. If it fails,
stop and report exactly which credential is missing or invalid — do not proceed
with partial access.

## Service table

| Service     | Skill            | Env var(s)                          | Config file                  | Preflight check |
|-------------|------------------|-------------------------------------|------------------------------|-----------------|
| GitHub      | github-repo      | `GH_TOKEN` (else `gh` keyring)      | `gh` CLI auth                | `gh auth status` |
| Railway     | railway-deploy   | `RAILWAY_TOKEN`                     | `railway` CLI login          | `railway whoami` |
| Cloudflare  | cloudflare-setup | `CLOUDFLARE_API_TOKEN`              | `~/.config/cloudflare/token` | `GET /user/tokens/verify` |
| reg.ru      | regru-domain     | `REGRU_USERNAME` + `REGRU_PASSWORD` | `~/.config/regru/credentials`| `domain/nop` |
| Yandex      | yandex-metrica   | `YANDEX_METRIKA_TOKEN`              | `~/.config/yandex/token`     | `GET /management/v1/counters` |

### How to obtain each credential

- **GitHub:** already handled by `gh auth login`. A fine-grained `GH_TOKEN` with
  `repo` scope also works.
- **Railway:** `railway login` (browserless: `railway login --browserless`), or a
  project/account token from <https://railway.com/account/tokens> exported as
  `RAILWAY_TOKEN`.
- **Cloudflare:** create an API token at
  <https://dash.cloudflare.com/profile/api-tokens> with permissions
  *Zone:Read, Zone:Edit, DNS:Edit, Zone Settings:Edit* and (for adding sites)
  *Account:Cloudflare Zones:Edit*. Export as `CLOUDFLARE_API_TOKEN`.
- **reg.ru:** API v2 uses your account login + password. Enable API access and
  (recommended) whitelist the caller IP in the reg.ru control panel. Set
  `REGRU_USERNAME` / `REGRU_PASSWORD`.
- **Yandex Metrica:** OAuth token with scope `metrika:write`. Register an app at
  <https://oauth.yandex.com/client/new> (scope Metrika), then obtain a token via
  the OAuth flow. Export as `YANDEX_METRIKA_TOKEN`.

## Config-file helper (resolution snippet)

Each skill uses this shape to resolve a single-value token:

```bash
resolve_token() {            # $1 = env var name, $2 = config file path
  local val="${(P)1:-}"      # zsh; in bash use: val="${!1:-}"
  [ -z "$val" ] && [ -f "$2" ] && val="$(head -n1 "$2")"
  printf '%s' "$val"
}
```

If the result is empty, the skill asks the user with `AskUserQuestion`.

## Money / irreversible actions

Any action that spends money or is hard to reverse (buying a domain, deleting a
zone or repo, taking a service offline) defaults to **dry-run + explicit
confirmation**. Skills print exactly what they would do and require a clear "yes"
before executing. The `new-service` orchestrator never performs a paid action on
its autonomous path.
