# new-service

Orchestrator skill: launch a brand-new web service end to end in one request.
Chains the five service skills, threading each step's outputs into the next:

```
github-repo → railway-deploy → cloudflare-setup → regru-domain → yandex-metrica
```

- **Inputs:** project name, domain, GitHub repo path, brief.
- **Does:** repo + landing → Railway deploy → Cloudflare zone (HTTPS + protection)
  + DNS → reg.ru nameservers → Yandex Metrica counter.
- **Aggregate preflight:** checks every credential up front and reports what's missing.
- **Gating:** never buys a domain or runs a paid action autonomously.
- **`--dry-run`:** prints the full ordered plan without side effects.

See [`references/flow.md`](references/flow.md) for the dependency diagram and
per-step contract. Credentials: [`../references/credentials.md`](../references/credentials.md).

## Install

```bash
ln -s "$PWD/new-service" ~/.claude/skills/new-service
```

Install the five sub-skills too (they're invoked by name):
`github-repo`, `railway-deploy`, `cloudflare-setup`, `regru-domain`, `yandex-metrica`.
