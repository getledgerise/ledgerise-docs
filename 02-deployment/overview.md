# deployment overview

Ledgerise is **customer-managed infrastructure**. You run it in your own cloud, VPS, or on-premise environment. Ledgerise does not host your transaction data, your database, or your credentials — those stay entirely within your control.

---

## what you need

Before deploying, have the following ready:

- A Linux server (cloud VM or on-premise) with at least 2 vCPU and 2 GB RAM.
- A **PostgreSQL database** — this is where all Ledgerise data is stored. Ledgerise does not provide or manage this database.
- Your **Ledgerise commercial license key and public key**, provided by email during onboarding.
- Credentials for your accounting system (for example, Zoho Books Client ID and Client Secret).
- A domain name or internal hostname for the Ledgerise dashboard and API, so you can set `VITE_API_BASE_URL` correctly.

---

## two deployment paths

### docker (recommended)

The primary commercial path. Ledgerise builds and maintains a versioned Docker image. You receive the image, configure a small set of environment variables, and run it with Docker Compose.

Commercial customers do not receive the source code or build context — only the compiled, versioned image. The image includes the API, web dashboard, and worker in a single container, split into three services by the Compose file.

→ [Docker deployment guide](docker-deployment.md)

### vps from source

A source-based deployment for internal demos, development environments, or operators who want to self-manage the build. This path requires Node.js 20, nginx, and systemd, and involves more manual steps.

> This path is not the primary commercial delivery. If you have a commercial license, use Docker.

→ [VPS deployment guide](vps-deployment.md)

---

## service architecture

Both deployment paths run three Ledgerise services:

| Service | Purpose | Default port |
|---|---|---|
| `api` | HTTP API, dashboard auth, health endpoint, adapters, posting | `3000` |
| `web` | Built React dashboard served by a static web server | `3001` |
| `worker` | Background runner for poll and journal engine jobs | — |

The three services share a single PostgreSQL database. They are stateless — the database is the source of truth for everything. This means you can restart any service at any time without data loss.

There is also a `migrate` tool service (Docker only) that applies database migrations. Migrations must run before the application starts, and again on every upgrade.

---

## what ledgerise controls, what you control

| You control | Ledgerise controls |
|---|---|
| The PostgreSQL database | The application image and releases |
| Encryption keys and secrets | The changelog and migration notes for each release |
| Adapter credentials | License key issuance and verification |
| Backups and restore drills | Implementation support for commercial customers |
| TLS and network perimeter | — |

---

## fresh deployments start in sandbox mode

A fresh deployment starts in **sandbox mode**. This is intentional and safe — sandbox mode means nothing you do can affect your accounting system or your production data. You use sandbox mode to configure adapters, sync your chart of accounts, build mapping rules, and import test transactions.

Once you are satisfied with your setup, you activate your commercial license in Settings → System. This switches the deployment to production and removes the Sandbox badge. Until you do that, no journal entries will ever be posted to your accounting system.

→ [Sandbox to production checklist](sandbox-to-production.md)

---

## where to go next

- [Docker deployment](docker-deployment.md) — the step-by-step guide for Docker Compose deployments
- [VPS deployment](vps-deployment.md) — for source-based installs
- [Environment variables](environment-variables.md) — complete reference for all configuration options
- [First login](first-login.md) — what to do immediately after deployment

[SCREENSHOT: Terminal output of `docker compose ps` showing `api`, `web`, and `worker` services all in `Up` state]
