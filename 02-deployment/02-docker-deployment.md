# docker deployment

This is the primary commercial deployment path. Ledgerise supplies a versioned Docker image. You run it with Docker Compose on your own server.

**Who this is for:** Admins performing a fresh installation.

---

## prerequisites

Before you begin, make sure you have:

- **Docker** and **Docker Compose** installed on the server.
- A **PostgreSQL 14+** database accessible from the server. This can be a managed database (AWS RDS, Supabase, etc.) or a self-managed instance.
- Your **Ledgerise image access credentials**, provided in your onboarding email.
- Your domain name or IP address, so you can set `VITE_API_BASE_URL`.
- The four required environment variable values listed in step 2.

---

## step 1 — pull the ledgerise image

Commercial customers receive a private registry URL and pull credentials in their onboarding email. Log in to the registry and pull the versioned image:

```bash
docker login registry.ledgerise.io
docker pull registry.ledgerise.io/ledgerise-cloud:<version>
```

Replace `<version>` with the version tag specified in your onboarding email or the latest release notification.

> The image includes the compiled API, web dashboard, worker, migrations, and operational scripts. It does not include TypeScript source files.

---

## step 2 — configure your environment file

The Compose file reads configuration from a `.env` file in the project directory. Copy the example file and fill in your values:

```bash
cp .env.example .env
nano .env
```

Four variables are required before the application can start:

| Variable | What to set |
|---|---|
| `DATABASE_URL` | Your PostgreSQL connection string, e.g. `postgresql://ledgerise:password@host:5432/ledgerise` |
| `AUTH_TOKEN_SECRET` | A strong random secret for signing session tokens. Generate one: `openssl rand -hex 64` |
| `LEDGERISE_CREDENTIALS_KEY` | A 64-character hex key for encrypting adapter credentials. Generate one: `openssl rand -hex 32` |
| `VITE_API_BASE_URL` | The public URL of your API, e.g. `https://api.your-domain.com` |

> **On `VITE_API_BASE_URL`:** This value is compiled into the browser bundle at build time, not read at runtime. For commercial customers using the pre-built image, Ledgerise builds this into the image during image preparation for your deployment. If you need a different API URL, contact Ledgerise — a rebuild is required.

→ Full reference: [environment variables](03-environment-variables.md)

---

## step 3 — run database migrations

Migrations must be applied before the application starts. Use the `migrate` tool service included in the Compose file:

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

[SCREENSHOT: Terminal showing `docker compose ps` output with `api`, `web`, and `worker` all in `Up` state]

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

[SCREENSHOT: Browser showing the `/healthcheck` JSON response with `"repository":"postgres"` and `"db":"ok"`]

---

## step 6 — sign in

Sign in with the default sandbox credentials:

- **Email:** `admin@ledgerise.dev`
- **Password:** `password`

The dashboard should load with a **Sandbox** badge in the top navigation bar. If you see this badge, your deployment is running correctly.

→ What to do next: [first login](04-first-login.md)

---

## troubleshooting

### dashboard shows "failed to fetch"

The frontend was built with the wrong or missing `VITE_API_BASE_URL`. This value is compiled into the browser bundle — changing `.env` after the image is built does not fix it. Contact Ledgerise for a rebuilt image with the correct API URL.

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
