# deployment overview

Ledgerise is **customer-managed infrastructure**. You run it in your own cloud or on-premise environment. Ledgerise does not host your transaction data, your database, or your credentials — those stay entirely within your control.

---

## what you need

Before deploying, have the following ready:

- A Linux server (cloud VM or on-premise) with at least 2 vCPU and 2 GB RAM.
- A **PostgreSQL database** — this is where all Ledgerise data is stored. Ledgerise does not provide or manage this database.
- Your **Ledgerise commercial license key**, provided during onboarding.
- Credentials for your accounting system (for example, Zoho Books Client ID and Client Secret).
- A domain name or internal hostname where users will open Ledgerise, for example `ledgerise.your-domain.com`.
- A server proxy or hosting panel route that forwards that hostname to the local Ledgerise proxy port.

---

## deployment

Ledgerise is deployed via Docker Compose. You use the public Ledgerise Docker image from GitHub Container Registry, configure a small set of environment variables, and run it with Docker Compose.

The image includes the services and operational tools needed to run Ledgerise. Your license key controls production activation.

→ [Docker deployment guide](02-docker-deployment.md)

---

## service architecture

The Ledgerise deployment runs four services:

| Service | Purpose | Default port |
|---|---|---|
| `caddy` | Bundled local proxy that routes dashboard and API traffic inside Docker | `18080` on localhost |
| `api` | HTTP API, dashboard auth, health endpoint, adapters, posting | `3000` |
| `web` | Built React dashboard served by a static web server | `3001` |
| `worker` | Background runner for poll and journal engine jobs | — |

The application services share a single PostgreSQL database. They are stateless — the database is the source of truth for everything. This means you can restart any service at any time without data loss.

There is also a `migrate` tool service (Docker only) that applies database migrations, and a seed command that initializes required built-in records for fresh databases. Migrations must run before the application starts, and again on every upgrade.

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

A fresh deployment starts in **sandbox mode**. This is intentional and safe — sandbox mode means nothing you do can affect your accounting system or your production data. You use sandbox mode to configure adapters, import your chart of accounts, build mapping rules, and import test transactions.

Once you are satisfied with your setup, you activate your commercial license in Settings → System. This switches the deployment to production and removes the Sandbox badge. Until you do that, no journal entries will ever be posted to your accounting system.

→ [Sandbox to production checklist](05-sandbox-to-production.md)

---

## where to go next

- [Docker deployment](02-docker-deployment.md) — the step-by-step guide for Docker Compose deployments
- [Environment variables](03-environment-variables.md) — complete reference for all configuration options
- [First login](04-first-login.md) — what to do immediately after deployment

![Docker Compose services running](../images/docker-compose-ps.png)
