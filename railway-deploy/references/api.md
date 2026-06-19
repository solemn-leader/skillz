# Railway reference

Primary interface is the `railway` CLI (v4+). The GraphQL API is a fallback for
fields the CLI doesn't surface as JSON.

## CLI command map

| Goal                              | Command |
|-----------------------------------|---------|
| Who am I (preflight)              | `railway whoami` |
| List projects/workspaces          | `railway list --json` |
| Create project                    | `railway init --name "$P" [--workspace "$W"] --json` |
| Add service from GitHub repo      | `railway add --service "$P" --repo "$OWNER/$NAME" --json` |
| Deploy local dir (alt)            | `railway up --service "$P" --ci` |
| Generate Railway domain           | `railway domain --service "$P" --json` |
| Attach custom domain (+DNS)       | `railway domain "$DOMAIN" --service "$P" --json` |
| Status                            | `railway status --json` |
| Logs                              | `railway logs --service "$P"` |
| Redeploy                          | `railway redeploy --service "$P"` |

All `--json` outputs are parsed with `jq`. Capture `projectId`, `domain`, and the
custom-domain CNAME target.

## Parsing examples

```bash
PROJECT_ID=$(railway init --name "$P" --json | jq -r '.projectId // .id')
RW_DOMAIN=$(railway domain --service "$P" --json | jq -r '.domain // .domains[0]')
# Custom domain: the CNAME target field name has varied; grab the first cname-ish value:
CNAME=$(railway domain "$DOMAIN" --service "$P" --json \
        | jq -r '.. | objects | (.cname // .value // empty)' | head -n1)
```

## GraphQL fallback

Endpoint: `https://backboard.railway.com/graphql/v2`
Auth header: `Authorization: Bearer $RAILWAY_TOKEN`
Content-Type: `application/json`

Create a custom domain and read its DNS target:

```bash
curl -s https://backboard.railway.com/graphql/v2 \
  -H "Authorization: Bearer $RAILWAY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation($i:CustomDomainCreateInput!){customDomainCreate(input:$i){id domain status{dnsRecords{hostlabel recordType requiredValue currentValue}}}}","variables":{"i":{"domain":"'"$DOMAIN"'","environmentId":"'"$ENV_ID"'","projectId":"'"$PROJECT_ID"'","serviceId":"'"$SERVICE_ID"'"}}}' \
  | jq '.data.customDomainCreate.status.dnsRecords'
```

`requiredValue` of the CNAME record is the target Cloudflare points at. IDs
(`environmentId`, `serviceId`) come from `railway status --json` or:

```graphql
query($id: String!) { project(id: $id) {
  services { edges { node { id name } } }
  environments { edges { node { id name } } }
} }
```

> Note: Railway has occasionally renamed JSON fields between CLI versions. Always
> prefer the CLI `--json` first; fall back to GraphQL only if the CLI output lacks
> the CNAME target. Verify field names against <https://docs.railway.com/reference/public-api>.
