# restore

This page covers restoring a Ledgerise PostgreSQL database from a backup — for disaster recovery, backup verification, or setting up a test environment from production data.

---

## when to restore

- **Disaster recovery** — the production database is lost, corrupted, or inaccessible and needs to be rebuilt from the most recent backup.
- **Backup verification** — a scheduled restore drill to confirm a backup is valid and the procedure works before you need it under pressure.
- **Test environment setup** — cloning production data into a staging or test environment to reproduce an issue or test a migration.

---

## restoring from a pg_dump backup

Use `pg_restore` for dumps created with `--format=custom`, or `psql` for plain SQL dumps.

**Using `pg_restore` (recommended — for custom-format dumps):**

```bash
# Create an empty target database first
createdb \
  --host=your-db-host \
  --username=postgres \
  ledgerise_restore

# Restore into it
pg_restore \
  --host=your-db-host \
  --port=5432 \
  --username=postgres \
  --dbname=ledgerise_restore \
  --no-owner \
  --role=ledgerise \
  /backups/ledgerise-20260630-020000.dump
```

If your backup was encrypted with GPG, decrypt it first:

```bash
gpg --decrypt /backups/ledgerise-20260630.dump.gpg | pg_restore \
  --host=your-db-host \
  --dbname=ledgerise_restore \
  --no-owner
```

**Using `psql` (for plain SQL dumps):**

```bash
psql \
  --host=your-db-host \
  --username=postgres \
  --dbname=ledgerise_restore \
  < /backups/ledgerise-20260630.sql
```

---

## after a restore: verification checklist

Once the restore completes, verify the data before pointing any live traffic at it.

**1. Start the Ledgerise API against the restored database.**

Update `DATABASE_URL` in your environment to point to the restored database, then start the API:

```bash
DATABASE_URL=postgres://ledgerise:password@your-db-host/ledgerise_restore \
  docker compose up api
```

**2. Confirm the healthcheck passes.**

```bash
curl https://ledgerise.your-domain.com/healthcheck
```

Expect: `"repository": "postgres"` and `"db": "ok"`. If the healthcheck fails, check the API logs for a database connection error — the most common cause is a `DATABASE_URL` pointing to the wrong host or using the wrong credentials.

**3. Spot-check record counts.**

In the Ledgerise dashboard, open the Transactions page, Journal Log, and Reconciliation pages and verify that record counts look reasonable for the backup date. You are looking for obvious signs that data is missing — for example, zero transactions when you expect thousands.

**4. Verify a known transaction end-to-end.**

Find a specific transaction reference you know existed before the backup and confirm it appears in the Transactions page with its expected mapping rule, journal entry, and reconciliation status. This confirms the restore is internally consistent, not just that tables exist.

---

## point-in-time recovery

If your PostgreSQL provider supports WAL archiving (AWS RDS, Google Cloud SQL, Supabase, Neon, and most managed providers do), you can restore to any specific timestamp within the archiving window — not just the last snapshot.

This is valuable when you need to recover from a specific event, such as an accidental bulk delete or a runaway migration, and you know the exact timestamp before which the data was correct.

Consult your database provider's documentation for the specific PITR restore procedure — it varies by provider. The post-restore verification steps above apply regardless of method.

---

## restore drill cadence

Run a restore drill against a non-production environment **at least once per quarter**. A drill should:

1. Take the most recent production backup.
2. Restore it to a separate database instance (not production).
3. Start a Ledgerise API pointed at the restored database.
4. Run the verification checklist above.
5. Document the time taken and any issues encountered.

The goal is to confirm that the backup is valid, that your team can execute the procedure, and that the expected recovery time is understood before it is needed in an actual incident.
