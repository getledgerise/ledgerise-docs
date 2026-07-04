# docker deployment

This is the primary commercial deployment path. Ledgerise publishes a versioned Docker image. You run it with Docker Compose on your own server.

**Who this is for:** Admins performing a fresh installation on a VPS or server that may already host other applications.

---

## how the docker install is exposed

Ledgerise uses one public hostname:

```text
https://ledgerise.your-domain.com
```

Inside Docker, Ledgerise runs four services:

- `caddy` — bundled local proxy for Ledgerise
- `api` — Ledgerise API and backend
- `web` — Ledgerise dashboard
- `worker` — background jobs

Only the bundled `caddy` service is exposed to the host, and only on localhost:

```text
127.0.0.1:18080
```

Your existing server proxy, hosting panel, nginx, Caddy, or Traefik should forward the Ledgerise hostname to that local port:

```text
https://ledgerise.your-domain.com -> http://127.0.0.1:18080
```

Then Ledgerise's bundled Caddy routes internally:

```text
/api/*       -> api container
/healthcheck -> api container
everything else -> dashboard
```

If your DNS already has a wildcard record, such as `*.your-domain.com -> VPS_PUBLIC_IP`, you may not need a new DNS record. You still need the server proxy route for the Ledgerise hostname.

---

## prerequisites

Before you begin, make sure you have:

- Docker and Docker Compose installed on the server.
- A PostgreSQL 14+ database accessible from the server.
- Your Ledgerise commercial license key, provided during onboarding.
- GitHub Container Registry access for the private Ledgerise image. Ledgerise provides the registry username `ledgerise-dev` and a personal access token during onboarding.
- A hostname where users will open Ledgerise, for example `ledgerise.your-domain.com`.
- A server proxy or hosting panel that can forward that hostname to `127.0.0.1:18080`.

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

x-ledgerise-app: &ledgerise-app
  image: ghcr.io/getledgerise/ledgerise:${LEDGERISE_IMAGE_TAG:-0.1.0}
  env_file:
    - .env
  environment:
    <<: *ledgerise-env
  restart: unless-stopped

services:
  caddy:
    image: caddy:2-alpine
    ports:
      - "127.0.0.1:${LEDGERISE_PROXY_PORT:-18080}:80"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
    restart: unless-stopped
    depends_on:
      api:
        condition: service_healthy
      web:
        condition: service_started

  api:
    <<: *ledgerise-app
    command: npm start -w apps/api
    healthcheck:
      test: ["CMD-SHELL", "node -e \"fetch('http://127.0.0.1:3000/healthcheck').then((r) => process.exit(r.ok ? 0 : 1)).catch(() => process.exit(1))\""]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 20s

  web:
    <<: *ledgerise-app
    command: node scripts/serve-web.mjs
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

Create `Caddyfile` in the same folder:

```caddyfile
:80 {
  encode zstd gzip

  handle /api/* {
    reverse_proxy api:3000
  }

  handle /healthcheck {
    reverse_proxy api:3000
  }

  handle {
    reverse_proxy web:3001
  }
}
```

Ledgerise publishes this image privately in GitHub Container Registry. Before pulling it, sign in to GHCR on the server with the username and personal access token provided by Ledgerise:

```bash
read -rsp "Ledgerise GHCR PAT: " LEDGERISE_GHCR_PAT
printf '\n'
printf '%s' "$LEDGERISE_GHCR_PAT" | docker login ghcr.io -u ledgerise-dev --password-stdin
unset LEDGERISE_GHCR_PAT
```

Paste the token when prompted; the terminal will not display it. Do not commit the personal access token to `docker-compose.yml`, `.env`, shell history, or deployment notes. The token is only for pulling the private image. Production use still requires a valid license key in Settings → System after first login.

---

## step 2 — configure your environment file

Create `.env` in `/opt/ledgerise` and fill in your values:

```env
LEDGERISE_IMAGE_TAG=0.1.0

LEDGERISE_DOMAIN=ledgerise.your-domain.com
LEDGERISE_PROXY_PORT=18080

DATABASE_URL=postgresql://ledgerise:change-me@your-db-host:5432/ledgerise
AUTH_TOKEN_SECRET=replace-with-openssl-rand-hex-64
LEDGERISE_CREDENTIALS_KEY=replace-with-openssl-rand-hex-32

RUN_RECONCILIATION_QUEUE_WORKER=true
```

Four variables are required before the application can start:

| Variable | What to set |
|---|---|
| `DATABASE_URL` | Your PostgreSQL connection string, e.g. `postgresql://ledgerise:password@your-db-host:5432/ledgerise` |
| `AUTH_TOKEN_SECRET` | A strong random secret for signing session tokens. Generate one: `openssl rand -hex 64` |
| `LEDGERISE_CREDENTIALS_KEY` | A 64-character hex key for encrypting adapter credentials. Generate one: `openssl rand -hex 32` |
| `LEDGERISE_DOMAIN` | Hostname only, without `https://` and without a path, e.g. `ledgerise.your-domain.com` |

`LEDGERISE_PROXY_PORT` defaults to `18080`. Change it only if another local service already uses that port.

Do not use `localhost` or `127.0.0.1` as the database host in `DATABASE_URL` for Docker deployments. Inside a Docker container, `localhost` means the container itself, not the VPS or database server. Use a hostname or IP address reachable from Docker, such as a managed database hostname, a private network IP, or the Docker service name if PostgreSQL runs in the same Compose network.

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

## step 4 — seed required built-in records

Fresh databases also need the built-in operator, adapter catalog, and default chart of accounts before the API can start. Run the seed command after migrations:

```bash
docker compose --profile tools run --rm api npm run seed:local
```

Run the seed command for fresh databases during initial setup. Do not use it as a routine upgrade step after operators have customized the default chart of accounts or built-in adapter metadata.

---

## step 5 — start the services

```bash
docker compose up -d caddy api web worker
```

Check that all four services are running:

```bash
docker compose ps
```

All four should show `Up` in the status column.

![Docker Compose services running](../images/docker-compose-ps.png)

---

## step 5 — connect your server proxy

Configure your existing server proxy or hosting panel so the Ledgerise hostname forwards to the local Ledgerise proxy port:

```text
https://ledgerise.your-domain.com -> http://127.0.0.1:18080
```

If you use nginx directly, the proxy block looks like this:

```nginx
server {
    server_name ledgerise.your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:18080;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Use your hosting panel, nginx, Caddy, Traefik, or Certbot flow to enable HTTPS for the hostname.

---

## step 6 — verify the health check

First verify the local Ledgerise proxy:

```bash
curl http://127.0.0.1:18080/healthcheck
```

Then verify the public hostname:

```bash
curl https://ledgerise.your-domain.com/healthcheck
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

Check the browser runtime config:

```bash
curl https://ledgerise.your-domain.com/runtime-config.js
```

It should contain your Ledgerise domain:

```js
window.__LEDGERISE_CONFIG__ = {"apiBaseUrl":"https://ledgerise.your-domain.com"};
```

---

## step 7 — sign in

Open the dashboard URL in your browser:

```text
https://ledgerise.your-domain.com
```

Sign in with the default sandbox credentials:

- **Email:** `admin@ledgerise.dev`
- **Password:** `password`

The dashboard should load with a **Sandbox** badge in the top navigation bar. If you see this badge, your deployment is running correctly.

→ What to do next: [first login](04-first-login.md)

After first login, activate your license from Settings → System using your Ledgerise license key. You do not need a license key in `.env`.

---

## troubleshooting

### dashboard shows "failed to fetch"

Check the runtime config your browser receives:

```bash
curl https://ledgerise.your-domain.com/runtime-config.js
```

The response should contain your Ledgerise domain:

```js
window.__LEDGERISE_CONFIG__ = {"apiBaseUrl":"https://ledgerise.your-domain.com"};
```

If it is wrong, update `LEDGERISE_DOMAIN` in `.env` and restart the web service:

```bash
docker compose restart web
```

If it is correct, check that your server proxy forwards `https://ledgerise.your-domain.com` to `http://127.0.0.1:18080`.

### public hostname does not load

Check the local Ledgerise proxy:

```bash
curl http://127.0.0.1:18080/healthcheck
```

If the local check works but the public hostname fails, the problem is in DNS, HTTPS, or the existing server proxy configuration.

### api shows `"repository":"memory"` in healthcheck

The API is not receiving `DATABASE_URL`, or `DATABASE_URL` points to a database host that the Docker container cannot reach. Do not use `localhost` or `127.0.0.1` for the database host unless PostgreSQL is running inside the same container, which is not the standard Ledgerise deployment. Use a reachable database hostname, private IP, or Compose service name. Restart the API service after fixing the value:

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
