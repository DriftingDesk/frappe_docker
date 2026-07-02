# Deploy ERPNext on Coolify

Custom image from `DriftingDesk/erpnextbuild1` (version-15) at **https://erpdemo.driftingdesk.com**.

## Prerequisites

- Coolify server with 8GB+ RAM
- DNS: `erpdemo.driftingdesk.com` A record → Coolify server IP
- GHCR image built: `ghcr.io/driftingdesk/erpnext-client:v15.114.0`

## Phase 2 — Build custom image (GitHub Actions)

1. Delete the failed Coolify app that pointed at `erpnextbuild1:master`.
2. GitHub → `DriftingDesk/frappe_docker` → **Actions** → **Build ERPNext Client Image** → **Run workflow**.
3. Wait until all steps are green, including **Smoke test image contents**.
4. GitHub → **Packages** → ensure `erpnext-client` is visible; if private, add `ghcr.io` registry credentials in Coolify.

## Phase 3 — Coolify Docker Compose deploy

1. **+ New Resource** → **Docker Compose**
2. Repository: `DriftingDesk/frappe_docker`, branch `main`
3. Compose file: `docker-compose.coolify.yml`
4. Environment variables (see `coolify.env.example`):

| Variable | Value |
|----------|-------|
| `ERPNEXT_VERSION` | `v15.114.0` |
| `CUSTOM_IMAGE` | `ghcr.io/driftingdesk/erpnext-client` |
| `CUSTOM_TAG` | `v15.114.0` |
| `PULL_POLICY` | `always` |
| `DB_PASSWORD` | strong random (Coolify auto-generate) |
| `FRAPPE_SITE_NAME_HEADER` | `erpdemo.driftingdesk.com` |
| `GUNICORN_WORKERS` | `2` |
| `GUNICORN_THREADS` | `4` |

5. Attach domain `erpdemo.driftingdesk.com` to service **`frontend`**, port **8080**, enable HTTPS.
6. **Deploy**. Wait for `configurator` to complete and all services healthy.
7. Confirm MariaDB port **3306 is not** published publicly.

## Phase 4 — Create site

Exec into the **backend** container (Coolify terminal or SSH):

```bash
bench new-site erpdemo.driftingdesk.com \
  --mariadb-user-host-login-scope='%' \
  --db-root-password '<DB_PASSWORD from Coolify>' \
  --admin-password '<strong admin password>' \
  --install-app erpnext
```

Site name must match the domain exactly.

## Phase 5 — Demo readiness

1. Open https://erpdemo.driftingdesk.com
2. Login: `Administrator` / your admin password
3. Complete ERPNext setup wizard (company, country, fiscal year)
4. Smoke-test: create a test Customer or Item in Desk

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Image pull failed | Add GHCR credentials in Coolify or make package public |
| `configurator` fails | Check `DB_PASSWORD` and MariaDB `db` service logs |
| 502 after deploy | Wait for `backend` to start; check `bench new-site` was run |
| Wrong site | `FRAPPE_SITE_NAME_HEADER` must match site name and domain |

## Rebuild image after fork updates

1. Push changes to `erpnextbuild1` `version-15`
2. Re-run **Build ERPNext Client Image** workflow
3. Redeploy in Coolify (same `CUSTOM_TAG` or bump tag in workflow + env)
