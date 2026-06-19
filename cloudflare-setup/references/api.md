# Cloudflare API v4 reference

Base: `https://api.cloudflare.com/client/v4`
Auth: `-H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"`
Every response is enveloped: `{"success":bool,"errors":[...],"result":...}`. Always
check `.success` and surface `.errors[].message` on failure.

## Token permissions needed

Zone:Read, Zone:Edit, DNS:Edit, Zone Settings:Edit, and Account → Cloudflare
Zones:Edit (to create new zones). Create at
<https://dash.cloudflare.com/profile/api-tokens>.

## Calls

### Verify token (preflight)
```bash
GET /user/tokens/verify
# -> .result.status == "active"
```

### Account id
```bash
GET /accounts
# -> .result[0].id
```

### Create zone
```bash
POST /zones
{"name":"$DOMAIN","account":{"id":"$ACCOUNT_ID"},"type":"full"}
# -> .result.id (zone id), .result.name_servers (assigned NS), .result.status
```

### Look up existing zone
```bash
GET /zones?name=$DOMAIN
# -> .result[0].id, .result[0].name_servers
```

### Settings (one PATCH per key)
```bash
PATCH /zones/$ZONE/settings/ssl                       {"value":"full"}
PATCH /zones/$ZONE/settings/always_use_https          {"value":"on"}
PATCH /zones/$ZONE/settings/automatic_https_rewrites  {"value":"on"}
PATCH /zones/$ZONE/settings/security_level            {"value":"medium"}
```

Loop:
```bash
apply_setting() {  # $1=key $2=json-value
  curl -s -X PATCH "https://api.cloudflare.com/client/v4/zones/$ZONE/settings/$1" \
    -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" -H "Content-Type: application/json" \
    -d "{\"value\":$2}" | jq -e '.success' >/dev/null || echo "WARN: $1 not applied"
}
apply_setting ssl '"full"'
apply_setting always_use_https '"on"'
apply_setting automatic_https_rewrites '"on"'
apply_setting security_level '"medium"'
```

### Bot Fight Mode (free tier)
```bash
PUT /zones/$ZONE/bot_management   {"fight_mode":true}
```

### DNS records
```bash
# create
POST /zones/$ZONE/dns_records
{"type":"CNAME","name":"@","content":"$TARGET","proxied":true,"ttl":1}
# list (to find existing)
GET /zones/$ZONE/dns_records?type=CNAME&name=$DOMAIN
# update existing
PUT /zones/$ZONE/dns_records/$RECORD_ID
{"type":"CNAME","name":"@","content":"$TARGET","proxied":true,"ttl":1}
```

Notes:
- `name:"@"` = apex; Cloudflare flattens apex CNAMEs automatically.
- `ttl:1` means "automatic". `proxied:true` = orange cloud (HTTPS/CDN/protection).
- Idempotency: list first; if a record with the same `type`+`name` exists, `PUT`
  it rather than `POST` a duplicate.

## Docs
- Zones: <https://developers.cloudflare.com/api/resources/zones/>
- DNS records: <https://developers.cloudflare.com/api/resources/dns/>
- Zone settings: <https://developers.cloudflare.com/api/resources/zones/subresources/settings/>
