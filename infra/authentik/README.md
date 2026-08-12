# Authentik IdP — deployment (ENG-327)

Self-hosted Authentik for teachers/staff identity (Story 00). All LDS configuration is
declarative in `blueprints/lds-identity.yaml` — the worker applies it automatically on
boot and re-applies it when the file changes. Authentik itself stays **stock**: upgrading
is bumping `AUTHENTIK_TAG`.

Sizing (measured in ENG-325): ~880 MB RAM idle for the full stack — a 2 vCPU / 4 GB VPS
is enough. This compose is designed to fold into the platform-wide compose later
(architecture: single VPS running intranet + Postgres + Authentik + Garage + loader).

## Prerequisites

1. **DNS**: an A record for the IdP host, e.g. `auth.libreriadesatoshi.com`.
2. **TLS / reverse proxy**: terminate HTTPS in front (Caddy/nginx/Traefik) and proxy to
   `server:9000`. Authentik must be reached over HTTPS in production (OAuth callbacks).
3. **OAuth apps** (callback URLs use the final hostname):
   - GitHub → Settings → Developer settings → OAuth Apps:
     callback `https://<host>/source/oauth/callback/github/`
   - Google Cloud Console → Credentials → OAuth client (Web):
     redirect `https://<host>/source/oauth/callback/google/`

## Deploy

```bash
cp .env.example .env        # fill every change-me (openssl rand -base64 48 / rand -hex 32)
docker compose up -d
# wait for http://<host>/-/health/ready/ → 200
```

First boot creates `akadmin` (password/token from `AUTHENTIK_BOOTSTRAP_*` — no browser
wizard) and the worker applies `lds-identity`, which configures:

- GitHub + Google sources with `email_link` (auto-link on matching verified email —
  Flow 1 of `docs/design/00-identity-linking.md`), shown on the login screen.
- Local username/password accounts (stock default flow; sources are additive).
- Source enrollment writes users as type **internal** (spike gotcha: stock default is
  external, which locks new users out of the user UI).
- `github_id` scope mapping (Flow 3 claim) and the `lds-intranet` OIDC provider +
  application (grant types explicit, `sub_mode: hashed_user_id`) ready for ENG-328.

## Verify

```bash
source .env
H="Authorization: Bearer $AUTHENTIK_BOOTSTRAP_TOKEN"
B=https://<host>/api/v3
curl -s -H "$H" "$B/managed/blueprints/?search=lds-identity" | jq '.results[0].status'   # "successful"
curl -s -H "$H" "$B/sources/all/" | jq '[.results[].slug]'                               # github, google
```

Login screen shows GitHub + Google buttons; `email/username + password` keeps working.

## Operations

- **Backup:** nightly `docker compose exec postgresql pg_dump -U authentik authentik`
  (all state incl. users lives in Postgres; media/certs under `./data` and `./certs`).
- **Upgrade:** bump `AUTHENTIK_TAG` in `.env` → `docker compose up -d`. Blueprints
  re-apply; nothing is hand-configured.
- **Config change:** edit the blueprint, commit, redeploy the file — never click it into
  the UI (drift). The worker re-applies changed blueprints automatically (~minutes) or
  force with: Admin → Customization → Blueprints → Apply.
- **Pending event notification (Flow 1):** configure a notification rule on the
  `source_linked` event → email transport, once SMTP settings exist for the deployment
  (needs the platform's SMTP account; tracked as a follow-up in ENG-330).

## Notes

- The `AUTHENTIK_BOOTSTRAP_*` variables only act on the very first boot (empty DB);
  rotate the token afterwards if it leaves the machine.
- The worker mounts `docker.sock` (official compose default, used for outposts); we run
  no outposts — remove the mount if the host's security posture requires it.
