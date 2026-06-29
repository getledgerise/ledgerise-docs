# vps deployment

This guide covers deploying Ledgerise from source on a Linux VPS. This path is intended for **internal demos, development environments, and operators who prefer a source-based install**.

> **Commercial customers:** Use the [Docker deployment](docker-deployment.md) path instead. The Docker image is the supported commercial delivery. This path is provided as a reference and is not covered by commercial implementation support.

---

## prerequisites

- Ubuntu 22.04 LTS or Debian 12 (other Linux distributions supported, commands may vary)
- A domain name or IP address for the server
- A PostgreSQL 14+ database on the server or accessible remotely
- `sudo` access on the server

---

## step 1 — install node.js 20, postgresql, and nginx

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs postgresql postgresql-contrib nginx
node -v
psql --version
```

Confirm Node.js 20 and PostgreSQL are both installed before proceeding.

---

## step 2 — create the database

```bash
sudo -u postgres psql <<'SQL'
CREATE USER ledgerise WITH PASSWORD 'change-me';
CREATE DATABASE ledgerise OWNER ledgerise;
SQL
```

Replace `change-me` with a strong password. Use only alphanumeric characters in the password — characters like `@`, `#`, `?`, and `&` can corrupt the `DATABASE_URL` connection string unless properly URL-encoded.

If you are on PostgreSQL 15 or later, grant schema permissions:

```bash
sudo -u postgres psql -d ledgerise -c "GRANT ALL ON SCHEMA public TO ledgerise;"
```

---

## step 3 — clone the repository and build

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

Replace `https://api.your-domain.com` with the public URL where your API will be accessible. This value is compiled into the browser bundle — it cannot be changed after the build without rebuilding.

---

## step 4 — configure the environment file

Create the `.env` file, owned by `www-data`, with restricted permissions:

```bash
sudo touch /opt/ledgerise/.env
sudo chown www-data:www-data /opt/ledgerise/.env
sudo chmod 600 /opt/ledgerise/.env
sudo -u www-data nano /opt/ledgerise/.env
```

Add the required environment variables:

```env
DATABASE_URL=postgresql://ledgerise:change-me@localhost:5432/ledgerise
AUTH_TOKEN_SECRET=<output of: openssl rand -hex 64>
LEDGERISE_CREDENTIALS_KEY=<output of: openssl rand -hex 32>
VITE_API_BASE_URL=https://api.your-domain.com
RUN_RECONCILIATION_QUEUE_WORKER=true
```

→ Full reference: [environment variables](environment-variables.md)

---

## step 5 — run migrations

```bash
cd /opt/ledgerise
sudo -u www-data node --env-file=/opt/ledgerise/.env scripts/run-migrations.mjs
```

This applies all pending SQL migrations. Always run this before starting the services, and again after every upgrade.

---

## step 6 — create systemd service files

Find the Node.js binary path first:

```bash
which node
```

Create `/etc/systemd/system/ledgerise-api.service`. Replace `<node-path>` with the output of `which node`:

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

Enable and start both services:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ledgerise-api ledgerise-worker
sudo systemctl status ledgerise-api
```

---

## step 7 — serve the dashboard with nginx

Copy the built web assets to the nginx web root:

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

Enable the site and reload nginx:

```bash
sudo ln -s /etc/nginx/sites-available/ledgerise /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

[SCREENSHOT: nginx configuration file in a terminal editor with the `proxy_pass` and `try_files` directives visible]

---

## step 8 — add tls with certbot

```bash
sudo apt-get install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

Certbot will obtain a certificate and update your nginx configuration automatically. Test that HTTPS is working before continuing.

---

## step 9 — verify the deployment

```bash
curl http://localhost:3000/healthcheck
```

The response should include `"repository":"postgres"` and `"db":"ok"`. If it shows `"repository":"memory"`, the API is not reading `DATABASE_URL` — check the `.env` file and restart the service.

Open your domain in a browser. You should see the Ledgerise login screen.

→ What to do next: [first login](first-login.md)

---

## upgrading a vps deployment

When a new version is available:

```bash
cd /opt/ledgerise
git pull origin main
sudo -u www-data npm install
sudo -u www-data VITE_API_BASE_URL=https://api.your-domain.com npm run build
sudo -u www-data node --env-file=/opt/ledgerise/.env scripts/run-migrations.mjs
sudo cp -r /opt/ledgerise/apps/web/dist /var/www/ledgerise
sudo systemctl restart ledgerise-api ledgerise-worker
sudo nginx -t && sudo systemctl reload nginx
```

→ See also: [upgrading](upgrading.md)

---

## troubleshooting

### dashboard shows "failed to fetch"

`VITE_API_BASE_URL` was missing or incorrect when the web bundle was built. The value is compiled in at build time — you must rebuild:

```bash
cd /opt/ledgerise
sudo -u www-data VITE_API_BASE_URL=https://api.your-domain.com npm run build
sudo cp -r /opt/ledgerise/apps/web/dist /var/www/ledgerise
sudo systemctl reload nginx
```

### api not reading environment variables

If the API starts in memory mode, verify the `.env` file exists, is owned by `www-data`, and contains a valid `DATABASE_URL`. Then restart:

```bash
sudo systemctl restart ledgerise-api
```

### `relation "recon_jobs" does not exist`

Migrations did not run or did not complete. Run them again:

```bash
sudo -u www-data node --env-file=/opt/ledgerise/.env scripts/run-migrations.mjs
```
