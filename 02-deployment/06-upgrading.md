# upgrading

When Ledgerise publishes a new version, you will receive an email with a changelog and migration notes. Review the changelog before upgrading — it will tell you whether the release contains breaking changes, schema migrations, or configuration changes that need your attention.

---

## before you upgrade

- Read the changelog for the release you are applying.
- Back up your database before upgrading. Migrations alter the schema and cannot be automatically rolled back. → [Backups](../10-data-management/01-backups.md)
- Plan for a brief service interruption. Users cannot access the dashboard while services are restarting.

---

## upgrading a docker deployment

This is the standard upgrade path for commercial customers:

```bash
# 1. Pull the new image
docker compose pull

# 2. Run migrations before restarting services
docker compose --profile tools run --rm migrate

# 3. Restart all services
docker compose up -d api web worker
```

Run the migration step before restarting — never the other way around. Starting the new API version before migrations are applied can cause the application to fail to start or behave incorrectly.

Verify the upgrade:

```bash
docker compose ps
curl http://localhost:3000/healthcheck
```

All three services should be `Up` and the health check should return `"db":"ok"`.

---

## after upgrading

- Check the dashboard loads correctly.
- Confirm the health check endpoint returns `"db":"ok"`.
- Check the Journal Log — verify the engine continues to run on its normal schedule.
- If the upgrade introduced any configuration changes noted in the changelog, apply them now.

---

## if something goes wrong

### migrations fail

Stop the services and restore your database backup before attempting again. Do not run the new application version against a schema that failed to migrate cleanly.

### api does not start after upgrade

Check the API logs for the error:

```bash
docker compose logs api
```

Common causes: a new required environment variable that is not yet set in `.env`, or a migration that did not complete.

### dashboard shows blank page or "failed to fetch"

Check the dashboard runtime config before assuming the image is wrong:

```bash
curl https://your-dashboard-domain/runtime-config.js
```

It should contain the public API URL. If it does not, update `PUBLIC_API_BASE_URL` in `.env` and restart the `web` service:

```bash
docker compose restart web
```
