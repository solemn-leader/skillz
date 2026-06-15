# new-service data flow

## Dependency diagram

```
                 ┌──────────────┐
                 │ github-repo  │  in: repo path, project name, brief
                 └──────┬───────┘  out: repo_url, clone_path
                        │ repo_url / repo path
                        ▼
                 ┌──────────────┐
                 │railway-deploy│  in: repo, custom_domain=domain
                 └──────┬───────┘  out: railway_domain, cname_target
                        │ cname_target
                        ▼
                 ┌──────────────┐
                 │cloudflare-   │  in: domain, CNAME @/www -> cname_target
                 │   setup      │  out: nameservers, zone_id
                 └──────┬───────┘
                        │ nameservers
                        ▼
                 ┌──────────────┐
                 │ regru-domain │  in: domain, set-ns = nameservers
                 └──────┬───────┘  out: NS confirmation
                        │ (repo clone_path)
                        ▼
                 ┌──────────────┐
                 │yandex-metrica│  in: domain, repo=clone_path
                 └──────────────┘  out: counter_id, commit_sha -> redeploy
```

## Why this order

- **Cloudflare before reg.ru:** Cloudflare assigns the nameservers only when the
  zone is created. reg.ru's `set-ns` needs those exact values.
- **Railway before Cloudflare DNS:** the proxied CNAME record points at Railway's
  `cname_target`, which only exists after the custom domain is attached in Railway.
- **Metrica last:** it commits into the repo, and (with a Railway GitHub-connected
  service) that push triggers the final redeploy — so analytics ships with the site.

## Step contracts

| Step             | Required inputs                          | Outputs threaded forward          |
|------------------|------------------------------------------|-----------------------------------|
| github-repo      | repo path, project name, brief           | `repo_url`, `clone_path`          |
| railway-deploy   | repo, `custom_domain`                    | `railway_domain`, `cname_target`  |
| cloudflare-setup | domain, `cname_target`, proxied          | `nameservers`, `zone_id`          |
| regru-domain     | domain, `nameservers`                    | NS confirmation                   |
| yandex-metrica   | domain, `clone_path`                     | `counter_id`, `commit_sha`        |

## Gating points

- **Domain must already be registered** before `regru-domain set-ns`. If not, stop
  and route the user through the gated `buy` action (explicit confirmation, spends
  money), then resume from Cloudflare.
- Any sub-skill preflight failure halts the pipeline with a clear "step X needs Y".

## Failure / resume

The steps are idempotent enough to resume:
- github-repo: if the repo exists, reuse it.
- railway-deploy: if project/service exists, reuse and just read the domains.
- cloudflare-setup: if the zone exists, fetch it; DNS records are upserted.
- regru-domain: `set-ns` is safe to re-run.
- yandex-metrica: injection is idempotent (one snippet per page).
So a halted run can be re-invoked with the same inputs.
