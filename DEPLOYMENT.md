# Ledgerise Deployment Guide

Ledgerise Commercial is customer-managed first. Clients receive a versioned Docker image from Ledgerise and run it in their own cloud, VPS, or on-premise infrastructure with operator-owned Postgres, secrets, backups, adapter credentials, accounting credentials, and AI provider keys.

Managed cloud is deferred. This guide covers only:

- Docker deployment: the primary commercial path.
- VPS deployment: a source-based reference path for internal demos, development, or self-managed installs.

## Runtime Shape

The Docker deployment runs three Ledgerise services from the same prebuilt image. Customers run the image; they do not receive the repository or build context.

| Service | Purpose | Default port |
|---|---|---|
| `api` | HTTP API, dashboard auth, health endpoint, adapters, reconciliation, posting | `3000` |
| `web` | Built React dashboard served by the bundled static web server | `3001` |
| `worker` | Background worker entrypoint for poll and engine jobs | none |

The compose file also includes a `migrate` tool service under the `tools` profile. It applies SQL migrations from the built Ledgerise image to the operator-owned database and records applied files in `schema_migrations`.

The distributed runtime image includes compiled JavaScript, package manifests, production dependencies, web build output, migrations, and operational scripts. It does not include the TypeScript source tree or repo build files. Compiled JavaScript can still be inspected by a determined customer, so do not treat the container image as cryptographic IP protection.

## Required Environment

Copy `.env.example` to `.env` and set production values:

```bash
cp .env.example .env
```

Required before startup:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | Operator-owned Postgres database connection string |
| `AUTH_TOKEN_SECRET` | HMAC secret for dashboard session tokens |
| `LEDGERISE_CREDENTIALS_KEY` | AES-256-GCM key for adapter and AI credential encryption |
| `VITE_API_BASE_URL` | Public API URL baked into the web build |

Fresh deployments start in sandbox mode. On first startup, Ledgerise creates `admin@ledgerise.dev` with password `password` when no users exist. Do not configure bootstrap admin credentials in the runtime environment for the normal onboarding flow.

Commercial license material is entered later in Settings > System. A valid license switches the deployment from sandbox to production and stores the license material encrypted in `system_settings`.

Optional runtime variables:

| Variable | Default | Purpose |
|---|---|---|
| `API_PORT` | `3000` | Host port mapped to the API container |
| `WEB_PORT` | `3001` | Host port mapped to the web container |
| `RUN_GENERIC_POLL_ON_START` | `false` | Run the generic poll adapter once on worker startup |
| `RUN_GENERIC_POLL_SCHEDULE` | `false` | Keep the generic poll scheduler running |
| `RUN_ENGINE_ON_START` | `false` | Run the journal engine once on worker startup |
| `RUN_RECONCILIATION_QUEUE_WORKER` | `true` | Keep polling the durable reconciliation queue |

## Docker Deployment

Ledgerise builds and publishes the commercial image. `VITE_API_BASE_URL` is compiled into the web bundle, so it must be set when Ledgerise builds the image:

```bash
VITE_API_BASE_URL=https://ledgerise-api.example.com docker compose build
```

Commercial customers normally skip this build step and pull the versioned image supplied by Ledgerise.

Apply migrations before starting the application:

```bash
docker compose --profile tools run --rm migrate
```

For local/trial environments only, seed the default local operator and chart of accounts:

```bash
docker compose --profile tools run --rm api npm run seed:local
```

For client deployments, provision the built-in local operator and adapter catalog intentionally. The runtime uses the seeded `local-operator` record internally; customers do not need to configure an operator id or slug in `.env`.

Start the services:

```bash
docker compose up -d api web worker
```

Check service health:

```bash
docker compose ps
curl http://localhost:3000/healthcheck
```

The API health response reports whether the app is using Postgres and whether the database is reachable. Worker health is currently process-supervisor based; scheduled worker metrics and a first-class worker health surface belong to the remaining Phase 21 monitoring work.

Health responses also include cached license status and the local current-month transaction usage summary after production activation. Ledgerise counts usage inside the customer deployment against the operator-owned database and refreshes that state at startup, immediately after license activation, and then at most once per day. The app reports counts and limits, not transaction records, amounts, customer identities, or other financial data.

Production activation automatically creates the deployment identifier used for summary-only license check-ins and stores it in `system_settings`. Customers do not configure a separate check-in token, deployment id, or app version in `.env`. Check-ins use the verified license material as proof, the check-in endpoint is built into the Ledgerise runtime image, and the reported app version comes from image/build metadata.

The API posts summary-only usage state to Ledgerise licensing infrastructure after production activation when the license cache refreshes. The payload contains client/deployment identifiers, app version, license status, read-only flag, usage period, transaction count, transaction limit, repository kind, health status, and checked-at timestamp only.

## Docker Upgrade

For a versioned image release:

```bash
docker compose pull
docker compose --profile tools run --rm migrate
docker compose up -d api web worker
```

For a source-built local image:

```bash
docker compose build
docker compose --profile tools run --rm migrate
docker compose up -d api web worker
```

## VPS Reference Deployment

The VPS path is not the primary commercial delivery path. Use it only when source-based deployment is intentionally preferred.

### 1. Install Node.js 20, PostgreSQL, and nginx

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs postgresql postgresql-contrib nginx
node -v
psql --version
```

### 2. Create the database

```bash
sudo -u postgres psql <<'SQL'
CREATE USER ledgerise WITH PASSWORD 'change-me';
CREATE DATABASE ledgerise OWNER ledgerise;
SQL
```

Use an alphanumeric database password unless it is properly URL-encoded. Characters like `@`, `#`, `?`, and `&` can corrupt `DATABASE_URL`.

### 3. Clone and build

```bash
cd /opt
sudo git clone https://github.com/getledgerise/ledgerise-cloud.git ledgerise
sudo chown -R www-data:www-data ledgerise
sudo -u www-data bash
cd /opt/ledgerise
npm install
VITE_API_BASE_URL=https://api.your-domain.com npm run build
exit
```

### 4. Configure environment

Create `/opt/ledgerise/.env`, owned by `www-data`, with the required variables from this guide:

```bash
sudo touch /opt/ledgerise/.env
sudo chown www-data:www-data /opt/ledgerise/.env
sudo chmod 600 /opt/ledgerise/.env
sudo -u www-data nano /opt/ledgerise/.env
```

### 5. Run migrations and seed local/demo data if needed

```bash
cd /opt/ledgerise
sudo -u www-data node --env-file=/opt/ledgerise/.env scripts/run-migrations.mjs
```

For local/demo installs only:

```bash
sudo -u www-data node --env-file=/opt/ledgerise/.env scripts/run-sql-file.mjs infra/seed/0001_local_operator_and_adapters.sql
sudo -u www-data node --env-file=/opt/ledgerise/.env scripts/run-sql-file.mjs infra/seed/0002_default_coa.sql
```

### 6. Create systemd services

Find the Node binary:

```bash
which node
```

Create `/etc/systemd/system/ledgerise-api.service`:

```ini
[Unit]
Description=Ledgerise API
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/ledgerise
ExecStart=<node-path> --env-file=/opt/ledgerise/.env apps/api/dist/index.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Create `/etc/systemd/system/ledgerise-worker.service`:

```ini
[Unit]
Description=Ledgerise Worker
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/ledgerise
ExecStart=<node-path> --env-file=/opt/ledgerise/.env apps/worker/dist/index.js
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable services:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ledgerise-api ledgerise-worker
sudo systemctl status ledgerise-api
```

### 7. Serve the web frontend with nginx

```bash
sudo cp -r /opt/ledgerise/apps/web/dist /var/www/ledgerise
```

Create `/etc/nginx/sites-available/ledgerise`:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    root /var/www/ledgerise;
    index index.html;

    location ~ ^/(api|healthcheck) {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/ledgerise /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Add TLS with Certbot:

```bash
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

## Health Checks

- `GET /healthcheck`
- `GET /api/health`

The API returns `200` when healthy. In Postgres mode it also probes the database and returns `503` when Postgres is unreachable.

Reconciliation operational metrics are available to authenticated dashboard users:

- `GET /api/reconciliation/metrics`

The response includes run totals, failed runs, run duration summaries, compared-record count, matched-record count, match rate, break totals, SLA breaches, and the current API process rolling report-generation duration summary. These metrics are summary-only and do not expose transaction rows or report artifact contents.

## Reconciliation Import Limits

The API processes statement preview/import requests synchronously. Keep these caps conservative in customer-managed deployments:

| Variable | Default | Purpose |
|---|---:|---|
| `RECON_IMPORT_MAX_FILE_BYTES` | `5242880` | Maximum CSV content size accepted for preview/import. |
| `RECON_IMPORT_MAX_ROWS` | `50000` | Maximum CSV data rows accepted for synchronous preview/import. |
| `RECON_IMPORT_MAX_REQUEST_BYTES` | `5373952` | Maximum declared JSON request body size accepted before the API reads the body. |

Requests above the declared request-body cap return `413`. CSV files above the file or row cap return a structured reconciliation error before full CSV parsing. Matching and report generation can run outside the API process through durable reconciliation jobs claimed by the worker.

## Reconciliation Worker Jobs

The worker can process long-running reconciliation matching and report-generation work outside the API process.

Run the durable reconciliation queue worker. The Docker worker service enables this by default, so this override is only needed if the operator has disabled it in `.env`:

```bash
RUN_RECONCILIATION_QUEUE_WORKER=true \
docker compose up -d worker
```

The dashboard uses the queued endpoints for operator-triggered run matching and report generation. The API can enqueue durable work through:

- `POST /api/reconciliation/runs/:runId/match-jobs`
- `POST /api/reconciliation/reports/jobs`
- `GET /api/reconciliation/jobs`
- `GET /api/reconciliation/jobs/:jobId`

The worker claims jobs from Postgres with row locking, processes matching/report generation, and marks jobs as succeeded, failed, or pending for retry without sending customer transaction data outside the deployment.

## Backup And Restore Drills

Ledgerise uses BYODB. The operator owns backup storage, restore testing, retention, and point-in-time recovery. Follow `docs/help-docs/DATA-MANAGEMENT.md` for backup/restore commands and retention guidance.

Minimum production drill:

1. Restore the latest backup into an isolated database.
2. Start `api` against the restored `DATABASE_URL`.
3. Confirm `/healthcheck` returns `200`.
4. Verify sample counts for transactions, journal entries, reconciliation runs, matches, breaks, reports, and audit events.

## First Login

1. Open the dashboard URL in a browser.
2. In sandbox mode, log in with the pre-filled default credentials: `admin@ledgerise.dev` / `password`.
3. Confirm the app chrome shows the Sandbox badge.
4. Use sandbox mode to import up to 50 demo transactions, configure adapters, and validate the operator workflow.
5. Before production activation, invite named admin/finance users from Settings > Users.

## Sandbox To Production Checklist

- [ ] Confirm `DATABASE_URL`, `AUTH_TOKEN_SECRET`, `LEDGERISE_CREDENTIALS_KEY`, and `VITE_API_BASE_URL` are configured for the real deployment.
- [ ] Log in as `admin@ledgerise.dev` / `password` while the deployment is still in sandbox mode.
- [ ] Invite named operator users and confirm at least one named admin can sign in.
- [ ] Configure adapter credentials, COA, mapping rules, report sources, and any AI provider settings needed for production.
- [ ] Use Settings > System > Reset sandbox data to clear demo transactions, journals, posting batches, reconciliation runs, reports, jobs, and related operational audit entries.
- [ ] Enter the Ledgerise commercial license key and public key in Settings > System.
- [ ] Confirm Settings > System switches to Production License and the app chrome no longer shows the Sandbox badge.
- [ ] Confirm `/healthcheck` reports `environment_mode: "production"` and a valid license state.
- [ ] Start real imports/posting only after the sandbox data reset and production activation are complete.

## Production Checklist

- [ ] `AUTH_TOKEN_SECRET` is a strong random value.
- [ ] `LEDGERISE_CREDENTIALS_KEY` is a 64-character hex value.
- [ ] Ledgerise commercial license key and public key are verified from Settings > System.
- [ ] `DATABASE_URL` points to operator-owned Postgres.
- [ ] Database backups and restore drills are owned by the operator.
- [ ] At least one named admin user exists before real production data starts.
- [ ] API is served behind TLS.
- [ ] Logs are captured by the container platform or journald.
- [ ] Health check URL is configured in the load balancer or process supervisor.
- [ ] Migrations run before new API versions start.
- [ ] `VITE_API_BASE_URL` matches the actual public API URL used for the frontend build.
- [ ] Reconciliation retention/archive policy is documented by the operator before production import volume starts.

## Current Phase 21 Gaps

- Internal license admin APIs/models live in the separate `ledgerise-admin` control-plane repo.
- License usage check-in is implemented as a summary-only outbound API call after Settings > System production activation.
- Large-file import limits and request-size backpressure are enforced for synchronous statement preview/import.
- Reconciliation run/report processing uses durable Postgres queue claiming for API-enqueued matching/report jobs.
- Reconciliation retention is operator-controlled and documented in `docs/help-docs/DATA-MANAGEMENT.md`; the app does not automatically purge evidence.
- Monitoring should poll `/healthcheck` for service/database health and `/api/reconciliation/metrics` for reconciliation run health. The API also writes structured JSON logs for requests, health failures, license state, and report generation duration.

## Troubleshooting

### API starts in memory mode

`DATABASE_URL` is not being read. Check that the environment file contains `DATABASE_URL`, that the process loads the environment file, and that no conflicting shell value is overriding it.

### API exits during startup

Check `DATABASE_URL` first. Passwords containing URL metacharacters must be encoded.

### `permission denied for schema public` during migrations

PostgreSQL 15+ restricts `CREATE TABLE` on the public schema to the database owner. Fix:

```bash
sudo -u postgres psql -d ledgerise -c "GRANT ALL ON SCHEMA public TO ledgerise;"
```

### Dashboard shows "failed to fetch"

The frontend was built with the wrong `VITE_API_BASE_URL`. Rebuild the web bundle with the correct public API URL.

### Dashboard shows `relation "recon_jobs" does not exist`

The database has not applied the reconciliation queue migration. Run migrations before starting or after upgrading:

```bash
npm run migrate
```

For Docker deployments:

```bash
docker compose --profile tools run --rm migrate
```

### First login fails

Fresh sandbox deployments should create `admin@ledgerise.dev` with password `password` when no users exist. Check whether the default user exists and has a password hash:

```bash
psql "$DATABASE_URL" -c "SELECT email, role, status, password_hash IS NOT NULL AS has_password FROM users ORDER BY created_at;"
```

If the table is empty, confirm migrations and operator provisioning ran successfully, then restart the API in sandbox mode so the default admin can be created.

### Forgot the default admin password

In sandbox mode, the default admin remains `admin@ledgerise.dev` with password `password`; the UI keeps that user's role fixed as Admin, status fixed as Active, and hides password reset.

After production activation, use another named admin to reset the affected user's password from Settings > Users. If all admins are locked out, perform a deliberate database recovery:

1. Stop the API temporarily or put the deployment in maintenance mode.
2. Generate a replacement password hash with the Ledgerise API image:

   ```bash
   docker compose run --rm api node -e "import('./apps/api/dist/lib/crypto.js').then(({ hashPassword }) => console.log(hashPassword(process.argv[1])))" 'new-strong-password'
   ```

3. Update the affected admin user:

   ```bash
   psql "$DATABASE_URL" -c "UPDATE users SET password_hash = 'PASTE_HASH_HERE', role = 'admin', status = 'active', updated_at = now() WHERE email = 'admin@example.com';"
   ```

4. Restart the API and sign in with the replacement password.
5. Immediately rotate the password again from the dashboard and review the audit log.

## Structured Logs

The API writes newline-delimited JSON to stdout. Each line includes fields such as timestamp, level, event, method, path, status, duration, and remote address.

Important events:

| Event | Level | Description |
|---|---|---|
| `server_start` | info | API process started |
| `http_request` | info | Every HTTP request |
| `auth_login` | info | Successful dashboard login |
| `auth_login_failed` | warn | Failed login attempt |
| `auth_logout` | info | Dashboard logout |
| `health_db_failed` | error | Health check DB probe failed |
| `credentials_decrypt_failed` | error | AES-GCM auth tag mismatch |
| `unhandled_request_error` | error | Unexpected exception in request handler |
