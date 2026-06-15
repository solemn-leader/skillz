# reg.ru API v2 reference

Base: `https://api.reg.ru/api/regru2`
Transport: HTTP POST, form-encoded. Common params on every call:

| Param                  | Value |
|------------------------|-------|
| `username`             | `$REGRU_USERNAME` (reg.ru account login) |
| `password`             | `$REGRU_PASSWORD` |
| `input_format`         | `json` |
| `output_content_type`  | `json` |
| `input_data`           | a JSON string with the method-specific args |

Response is JSON with a top-level `result` (`"success"` or `"error"`). On error,
read `error_code` / `error_text`. Per-domain results are under
`answer.domains[]` with each item's own `result`.

Always send `input_data` via `--data-urlencode` so the JSON is escaped correctly.

## Auth/IP note

reg.ru requires API access to be enabled for the account, and commonly an IP
allowlist. `ACCESS_DENIED_FROM_IP` / `NO_SUCH_USER` / `PASSWORD_AUTH_FAILED` mean
the credential or IP setup is wrong — fix in the reg.ru control panel.

## Methods used

### Connectivity / auth test
```
POST /domain/nop
input_data = {"domains":[{"dname":"$DOMAIN"}]}
-> result: "success"
```

### Check availability
```
POST /domain/check
input_data = {"domains":[{"dname":"$DOMAIN"}]}
-> answer.domains[].result in {avail, unavail, ...}
```

### Read nameservers
```
POST /domain/get_nss
input_data = {"domains":[{"dname":"$DOMAIN"}]}
-> answer.domains[].nss
```

### Set nameservers (point at Cloudflare)
```
POST /domain/update_nss
input_data = {
  "domains":[{"dname":"$DOMAIN"}],
  "nss":{"ns0":"<cf-ns-1>","ns1":"<cf-ns-2>"}
}
```
Cloudflare free plan assigns exactly two nameservers; map them to `ns0`/`ns1`.
For glue/extra NS, reg.ru also accepts `ns0_ip` etc. — not needed for Cloudflare.

### Register a domain (GATED — costs money)
```
POST /domain/create
input_data = {
  "domains":[{"dname":"$DOMAIN"}],
  "period": 1,
  "nss":{"ns0":"<cf-ns-1>","ns1":"<cf-ns-2>"},
  "contacts": { ...registrant/admin/tech/billing fields... }
}
```
`create` needs full contact data and account balance. Confirm price and contacts
with the user first; never auto-run. Exact contact field names per reg.ru docs.

## Docs
- API v2: <https://www.reg.ru/support/help/api2>
- Method list: <https://www.reg.ru/reseller/api2doc>
