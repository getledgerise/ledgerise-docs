# docker deployment

This is the primary commercial deployment path. Ledgerise publishes a versioned Docker image. You run it with Docker Compose on your own server.

**Who this is for:** Admins performing a fresh installation.

---

## prerequisites

Before you begin, make sure you have:

- **Docker** and **Docker Compose** installed on the server.
- A **PostgreSQL 14+** database accessible from the server. This can be a managed database (AWS RDS, Supabase, etc.) or a self-managed instance.
- Your **Ledgerise commercial license key**, provided during onboarding.
- Your domain name or IP address, so you can set `PUBLIC_API_BASE_URL`.
- The four required environment variable values listed in step 2.

---

## step 1 — create the deployment folder

Create one folder on the server for Ledgerise:

```bash
sudo mkdir -p /opt/ledgerise
sudo chown "$USER":"$USER" /opt/ledgerise
cd /opt/ledgerise
```

Create `docker-compose.yml` in that folder:

```yaml
name: ledgerise-cloud

x-ledgerise-env: &ledgerise-env
  NODE_ENV: production
  API_PORT: 3000

x-ledgerise-app: &ledgerise-app
  image: ghcr.io/getledgerise/ledgerise-cloud:${LEDGERISE_IMAGE_TAG:-0.1.0}
  env_file:
    - .env
  environment:
    <<: *ledgerise-env
  restart: unless-stopped

services:
  api:
    <<: *ledgerise-app
    command: npm start -w apps/api
    ports:
      - "${API_PORT:-3000}:3000"
    healthcheck:
      test: ["CMD-SHELL", "node -e \"fetch('http://127.0.0.1:3000/healthcheck').then((r) => process.exit(r.ok ? 0 : 1)).catch(() => process.exit(1))\""]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s

  web:
    <<: *ledgerise-app
    command: node scripts/serve-web.mjs
    ports:
      - "${WEB_PORT:-3001}:3001"
    depends_on:
      api:
        condition: service_healthy

  worker:
    <<: *ledgerise-app
    command: npm start -w apps/worker
    environment:
      <<: *ledgerise-env
      RUN_RECONCILIATION_QUEUE_WORKER: ${RUN_RECONCILIATION_QUEUE_WORKER:-true}
    depends_on:
      api:
        condition: service_healthy

  migrate:
    <<: *ledgerise-app
    profiles:
      - tools
    command: npm run migrate
    restart: "no"
```

The image is public, so `docker login` is not required. Production use still requires a valid license key in Settings → System after first login.

---

## step 2 — configure your environment file

Create `.env` in `/opt/ledgerise` and fill in your values:

```env
LEDGERISE_IMAGE_TAG=0.1.0

DATABASE_URL=postgresql://ledgerise:change-me@your-db-host:5432/ledgerise
AUTH_TOKEN_SECRET=replace-with-openssl-rand-hex-64
LEDGERISE_CREDENTIALS_KEY=replace-with-openssl-rand-hex-32
PUBLIC_API_BASE_URL=https://api.your-domain.com

API_PORT=3000
WEB_PORT=3001
RUN_RECONCILIATION_QUEUE_WORKER=true
```

Four variables are required before the application can start:

| Variable | What to set |
|---|---|
| `DATABASE_URL` | Your PostgreSQL connection string, e.g. `postgresql://ledgerise:password@host:5432/ledgerise` |
| `AUTH_TOKEN_SECRET` | A strong random secret for signing session tokens. Generate one: `openssl rand -hex 64` |
| `LEDGERISE_CREDENTIALS_KEY` | A 64-character hex key for encrypting adapter credentials. Generate one: `openssl rand -hex 32` |
| `PUBLIC_API_BASE_URL` | The public URL of your API, e.g. `https://api.your-domain.com` |

> **On `PUBLIC_API_BASE_URL`:** The Docker web service serves this value to browsers at runtime through `/runtime-config.js`. You can change the API URL by updating `.env` and restarting the `web` service; you do not need a rebuilt image just to change domains.

→ Full reference: [environment variables](03-environment-variables.md)

Generate the two required secrets on the server:

```bash
openssl rand -hex 64
openssl rand -hex 32
```

Paste the first output into `AUTH_TOKEN_SECRET`. Paste the second output into `LEDGERISE_CREDENTIALS_KEY`.

---

## step 3 — run database migrations

Migrations must be applied before the application starts. Use the `migrate` tool service included in the Compose file:

```bash
docker compose pull
```

```bash
docker compose --profile tools run --rm migrate
```

This applies all pending SQL migrations to your database and records them in `schema_migrations`. It is safe to run repeatedly — already-applied migrations are skipped.

> Always run migrations before starting the app, and run them again on every upgrade before restarting services.

---

## step 4 — start the services

```bash
docker compose up -d api web worker
```

This starts three services in the background:

- `api` — the HTTP API and dashboard backend
- `web` — the static React dashboard
- `worker` — the background runner for the journal engine and poll adapters

Check that all three are running:

```bash
docker compose ps
```

All three should show `Up` in the status column.

![Docker Compose services running](../images/docker-compose-ps.png)

---

## step 5 — verify the health check

```bash
curl http://localhost:3000/healthcheck
```

A healthy response looks like this:

```json
{
  "status": "ok",
  "repository": "postgres",
  "db": "ok"
}
```

If `repository` shows `"memory"` instead of `"postgres"`, the API is not reading your `DATABASE_URL`. See the troubleshooting section below.

Open the dashboard URL in your browser (`http://your-server:3001` or your configured domain). You should see the Ledgerise login screen.

![Healthcheck response](../images/healthcheck-response.png)

---

## step 6 — sign in

Sign in with the default sandbox credentials:

- **Email:** `admin@ledgerise.dev`
- **Password:** `password`

The dashboard should load with a **Sandbox** badge in the top navigation bar. If you see this badge, your deployment is running correctly.

→ What to do next: [first login](04-first-login.md)

After first login, activate your license from Settings → System using your Ledgerise license key. You do not need a license key in `.env`.

---

## troubleshooting

### dashboard shows "failed to fetch"

The web service may not be receiving the correct `PUBLIC_API_BASE_URL`, or your reverse proxy may be serving an old static dashboard instead of the Docker `web` service.

Check the runtime config your browser receives:

```bash
curl https://your-dashboard-domain/runtime-config.js
```

The response should contain your public API URL:

```js
window.__LEDGERISE_CONFIG__ = {"apiBaseUrl":"https://api.your-domain.com"};
```

If it is wrong, update `PUBLIC_API_BASE_URL` in `.env` and restart the web service:

```bash
docker compose restart web
```

### api shows `"repository":"memory"` in healthcheck

The API is not receiving `DATABASE_URL`. Check that the variable is correctly set in your `.env` file and that the Compose file is reading from that file. Restart the API service after fixing the value:

```bash
docker compose restart api
```

### `permission denied for schema public` during migrations

PostgreSQL 15+ restricts `CREATE TABLE` on the public schema. Grant the required permission to your database user:

```bash
psql "$DATABASE_URL" -c "GRANT ALL ON SCHEMA public TO ledgerise;"
```

Then run migrations again.

### first login fails

If the default admin user was not created, migrations may not have run successfully, or the API may have started before the database was ready. Confirm migrations ran cleanly, then restart the API:

```bash
docker compose restart api
```
