# environment variables

All Ledgerise configuration is passed through environment variables set in your `.env` file. This page documents every variable, what it controls, and whether it is required.

Copy `.env.example` to `.env` before editing:

```bash
cp .env.example .env
```

---

## required variables

These four variables must be set before the application can start. Missing any of them will prevent Ledgerise from starting or will cause it to fall back to unsafe defaults.

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string. Format: `postgresql://user:password@your-db-host:5432/dbname`. The API and worker both connect to this database from inside Docker. Do not use `localhost` or `127.0.0.1` unless PostgreSQL is running in the same container. |
| `AUTH_TOKEN_SECRET` | Secret used to sign and verify dashboard session tokens. Must be a strong random value. Generate one with `openssl rand -hex 64`. Rotate this value to invalidate all active sessions. |
| `LEDGERISE_CREDENTIALS_KEY` | AES-256-GCM encryption key for adapter credentials (API keys, OAuth tokens) and AI provider keys stored in the database. Must be a 64-character hex string. Generate one with `openssl rand -hex 32`. **This value cannot be changed after credentials have been stored without re-encrypting the existing data.** |
| `LEDGERISE_DOMAIN` | Hostname where users open Ledgerise, without `https://` and without a path — for example, `ledgerise.your-domain.com`. The Docker web service uses this to serve browser runtime config through `/runtime-config.js`. |

> **Treat `AUTH_TOKEN_SECRET` and `LEDGERISE_CREDENTIALS_KEY` as secrets.** Do not commit them to version control. Do not log them. Use a secrets manager or a `.env` file with restricted permissions (`chmod 600 .env`).

---

## optional — runtime behaviour

These variables tune how the services behave at runtime. Defaults are safe for most deployments.

| Variable | Default | Description |
|---|---|---|
| `LEDGERISE_IMAGE_TAG` | `0.1.0` | Docker image version used by `docker-compose.yml`, for example `0.1.0`. Change this when upgrading to a new Ledgerise release. |
| `LEDGERISE_PROXY_PORT` | `18080` | Local host port exposed by the bundled Ledgerise Caddy proxy. Point your existing server proxy to `http://127.0.0.1:18080`. |
| `RUN_ENGINE_ON_START` | `false` | If `true`, the worker runs one journal engine cycle immediately when it starts, before falling into its scheduled loop. Useful for local development. |
| `RUN_GENERIC_POLL_ON_START` | `false` | If `true`, the worker triggers one poll adapter cycle on startup. |
| `RUN_GENERIC_POLL_SCHEDULE` | `false` | If `true`, the worker keeps a recurring poll scheduler running. Set this if you are using the poll inbound adapter. |
| `RUN_RECONCILIATION_QUEUE_WORKER` | `true` | If `true`, the worker polls the durable reconciliation job queue and processes pending matching and report-generation jobs. Leave this at `true` in production — disabling it means reconciliation runs triggered from the dashboard will not be processed. |

---

## conditional — adapters and ai

Set these only if you are using the corresponding integration. You do not need to set all of them — only the ones for your chosen accounting system and AI provider.

| Variable | Description |
|---|---|
| `ZOHO_CLIENT_ID` | OAuth Client ID from your Zoho API console. Required for the Zoho Books outbound adapter. |
| `ZOHO_CLIENT_SECRET` | OAuth Client Secret for your Zoho app. Required for the Zoho Books outbound adapter. |
| `ZOHO_ORGANIZATION_ID` | Your Zoho Books Organization ID. Found in Zoho Books under Settings → Organisation Profile. |
| `AI_PROVIDER` | The AI provider to use for any AI-assisted features. Accepted values: `openai`, `anthropic`. |
| `AI_API_KEY` | API key for the configured `AI_PROVIDER`. This key belongs to you — Ledgerise uses it to make AI calls on your behalf and stores it encrypted using `LEDGERISE_CREDENTIALS_KEY`. |

---

## conditional — reconciliation import limits

These caps control how large a statement file the API will accept for synchronous preview and import. The defaults are suitable for most deployments. Lower them if you want to protect API memory on constrained servers.

| Variable | Default | Description |
|---|---|---|
| `RECON_IMPORT_MAX_FILE_BYTES` | `5242880` (5 MB) | Maximum CSV content size accepted for a statement preview or import request. |
| `RECON_IMPORT_MAX_ROWS` | `50000` | Maximum number of data rows accepted in a single statement import. |
| `RECON_IMPORT_MAX_REQUEST_BYTES` | `5373952` (≈5.1 MB) | Maximum declared JSON request body size. Requests above this limit receive a `413` response before the body is read. |

For deployments processing very large statement files — above 50,000 rows — reconciliation matching and report generation can run outside the API process through the durable worker queue. The `RUN_RECONCILIATION_QUEUE_WORKER` variable must be `true` for this to work.

---

## generating secrets

Use these commands to generate the required secret values:

```bash
# AUTH_TOKEN_SECRET — 128-character hex string
openssl rand -hex 64

# LEDGERISE_CREDENTIALS_KEY — 64-character hex string
openssl rand -hex 32
```

Run each command once and paste the output into your `.env` file. Do not share these values or commit them to version control.
