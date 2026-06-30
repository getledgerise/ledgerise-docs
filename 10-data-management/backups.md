# backups

Ledgerise uses a bring-your-own-database model: your PostgreSQL database is yours, and so is the responsibility for backing it up. Ledgerise has no access to your database and cannot perform or schedule backups on your behalf.

---

## what to back up

Everything Ledgerise stores is in your PostgreSQL database: transactions, journal entries, mapping rules, reconciliation runs, audit log entries, adapter configuration, and user accounts. There are no files outside the database that need to be backed up separately.

---

## recommended backup strategy

**Daily database snapshots** at minimum. For financial data, the commonly accepted baseline is:

- Daily full backups retained for 30 days
- Monthly backups retained for 12 months (to satisfy 7-year retention requirements when combined with archiving)

For high-volume operators where losing even an hour of transaction data is significant, add **WAL archiving for point-in-time recovery** (PITR). Most managed PostgreSQL providers (RDS, Cloud SQL, Supabase, Neon) offer this as a built-in feature. With WAL archiving enabled, you can restore to any point in time within the archiving window — not just to the last snapshot.

---

## taking a backup with pg_dump

For a VPS or self-managed PostgreSQL instance, use `pg_dump`:

```bash
pg_dump \
  --host=your-db-host \
  --port=5432 \
  --username=ledgerise \
  --dbname=ledgerise_prod \
  --format=custom \
  --file="/backups/ledgerise-$(date +%Y%m%d-%H%M%S).dump"
```

The `--format=custom` flag produces a compressed, portable dump file compatible with `pg_restore`. Store the output file in a location outside your database server — an object storage bucket (AWS S3, GCS, Backblaze B2) or an off-site NAS.

For a Docker deployment, run `pg_dump` from inside the database container or from a separate backup container with network access to the database:

```bash
docker compose exec db pg_dump \
  --username=postgres \
  --dbname=ledgerise_prod \
  --format=custom \
  > "/backups/ledgerise-$(date +%Y%m%d-%H%M%S).dump"
```

[SCREENSHOT: Terminal showing a pg_dump command completing successfully with the output file path and size]

---

## encrypt your backups

Any backup of a Ledgerise database should be encrypted before it is stored or transmitted, since the database contains financial records and audit trails even with PII masking in place.

A simple approach using GPG:

```bash
pg_dump ... --format=custom | gpg --symmetric --cipher-algo AES256 \
  > "/backups/ledgerise-$(date +%Y%m%d).dump.gpg"
```

Store the passphrase separately from the backup files — in a password manager or secrets vault, not on the same server.

---

## backup schedule

Automate backups using cron or your infrastructure's scheduling system. A daily backup at a low-traffic time (for example, 02:00 local time) is the baseline:

```
0 2 * * * /opt/scripts/ledgerise-backup.sh >> /var/log/ledgerise-backup.log 2>&1
```

Log the output of each backup run and alert on failures. A backup that ran silently and produced a corrupt file is as dangerous as no backup at all.

---

## test your backups

A backup you have never restored is not a backup. → See [restore](restore.md) for how to verify a backup is recoverable.

Run a restore drill at least once per quarter against a non-production environment. This confirms the backup file is valid and that your team knows the restore procedure before they need it under pressure.
