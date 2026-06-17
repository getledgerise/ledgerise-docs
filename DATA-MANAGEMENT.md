# Data Management: BYODB Backup, Restore, and Retention

Ledgerise is designed for Bring Your Own Database (BYODB) deployments. The operator provisions and controls the database, object storage, backup destination, retention policy, and encryption keys. This document is guidance for operating that database safely; it is not a Ledgerise-managed data custody policy.

The current commercial deployment model is on-premise Docker. If managed cloud is introduced later, the same principle must apply at the tenant boundary: data, adapter credentials, API keys, and AI provider keys remain operator-scoped. Ledgerise must store only encrypted credential material needed to run configured workflows and must not resell or proxy operator AI keys.

## Backup

### Logical backup (recommended for most deployments)

Use `pg_dump` against the operator-owned `DATABASE_URL` to take a consistent logical snapshot:

```bash
pg_dump "$DATABASE_URL" \
  --format=custom \
  --no-acl \
  --no-owner \
  --file="ledgerise-$(date +%Y%m%d%H%M%S).pgdump"
```

`--format=custom` produces a compressed, parallel-restorable file. Store the output in the operator's durable object storage account (S3, GCS, Azure Blob, on-prem backup storage, etc.).

### Continuous WAL archiving (for large or high-volume deployments)

For near-zero RPO, configure Postgres WAL archiving with a tool such as `pgbackup`, `WAL-G`, or `Barman`. Point `archive_command` at the operator-controlled object storage bucket. This enables point-in-time recovery (PITR).

### Recommended schedule

| Frequency | Method | Retention |
|---|---|---|
| Daily | `pg_dump` full logical backup | 30 days |
| Continuous | WAL archiving (if configured) | 7 days of WAL segments |

## Restore

### From a logical backup

```bash
createdb ledgerise_restored
pg_restore \
  --dbname="postgres://user:pass@host:5432/ledgerise_restored" \
  --no-acl \
  --no-owner \
  ledgerise-20260602120000.pgdump
```

Verify the restore before cutting over traffic:

```bash
psql "$RESTORED_DATABASE_URL" -c "SELECT count(*) FROM transactions;"
psql "$RESTORED_DATABASE_URL" -c "SELECT count(*) FROM journal_entries;"
```

### Testing restores

Run a restore drill monthly into an isolated environment. Verify:

1. All migrations applied cleanly (check `schema_migrations` or compare table counts).
2. API health check returns `200`.
3. A sample query across transactions, journal entries, and posting batches returns expected rows.

## Data Retention Guidance

Ledgerise should not decide when an operator's financial records are destroyed. The table below is a recommended minimum retention baseline for operator-owned databases, based on accounting audit expectations. Operators may keep data longer or implement stricter retention according to local law, customer contracts, and internal policy.

| Table | Recommended minimum retention | Notes |
|---|---|---|
| `transactions` | 7 years | Source-of-truth for ingested financial events; required for audit |
| `journal_entries` | 7 years | Accounting records; required for financial audit and reconciliation |
| `posting_batches` / `posting_artifacts` | 7 years | Evidence of what was posted to the general ledger and when |
| `recon_runs`, `recon_external_records`, `recon_cases`, `recon_matches`, `recon_breaks` | 7 years | Reconciliation evidence linking internal transactions to external processor/bank statements |
| `recon_reports` / `recon_report_downloads` | 7 years | Generated PDF/CSV evidence packages and artifact access audit |
| `recon_audit_log`, `recon_rule_audit` | 7 years | Append-only reconciliation event and rule-governance history |
| `recon_rules`, `recon_fee_schedules`, `recon_field_mappings`, `recon_report_sources` | Retain active records and historical versions for 7 years after retirement | Needed to explain how historical reconciliation outcomes were produced |
| `ingestion_errors` | 1 year | Operational diagnostics; can be pruned after resolution |
| `poll_runs` / `poll_cursors` | 90 days | Adapter operations log; short-lived operational data |
| `api_keys` | Retain metadata until revoked + 1 year | API key secrets are stored only as hashes; retain lifecycle metadata for audit |
| `users` | Retain until deactivated + 7 years | Access control audit trail |

### Reconciliation evidence retention

Reconciliation data is audit evidence, not temporary import scratch data. Retain enough of the following to reproduce why a transaction matched, broke, or was resolved:

- Imported statement metadata from `recon_runs`.
- Parsed external rows from `recon_external_records`.
- Case, match, and break state from `recon_cases`, `recon_matches`, and `recon_breaks`.
- Generated evidence artifacts from `recon_reports`, including checksums and PDF/CSV bytes.
- Download history from `recon_report_downloads`.
- Rule, fee schedule, mapping, and report-source definitions used at the time.
- Event history from `recon_audit_log` and `recon_rule_audit`.

Recommended policy:

- Keep live reconciliation data for at least the current financial year plus one prior year.
- Archive older reconciliation evidence to operator-controlled cold storage or an archive schema before pruning live rows.
- Keep generated report artifacts and checksums together. Do not keep only the PDF without the CSV evidence rows, or only the metadata without the artifact bytes.
- Never prune `recon_audit_log` or `recon_rule_audit` for retained reconciliation records; otherwise the evidence trail becomes incomplete.

If an operator decides to prune old reconciliation runs, delete by `operator_id` and run date after exporting and verifying the archive. Because several child tables cascade from `recon_runs`, test the exact delete in a restored copy first:

```sql
-- Example only: run in a restored/staging database before production.
BEGIN;

SELECT count(*)
FROM recon_runs
WHERE operator_id = :operator_id
  AND statement_date_to < DATE '2019-01-01';

-- Export/archive matching rows before deleting.
-- DELETE FROM recon_runs
-- WHERE operator_id = :operator_id
--   AND statement_date_to < DATE '2019-01-01';

ROLLBACK;
```

Do not prune reconciliation rows that support open breaks, unresolved disputes, ongoing audits, chargebacks, regulatory requests, or financial statements that are still within the operator's statutory retention window.

### Archival approach

Rather than deleting old rows, move aged rows to a separate archive schema or table partition, or export them to Parquet/CSV in operator-controlled cold storage before pruning. Keep at least a count and date-range summary in the live database to satisfy audit queries.

### No automatic purge

Ledgerise does not automatically delete financial or audit data. Retention and archival are operator decisions implemented as external scheduled jobs, database-level policies, or managed database lifecycle rules. The application should expose enough metadata to support those policies, but it should not silently purge records.

## Key Ownership

Ledgerise uses BYOK for AI provider keys and operator-managed secrets for deployment. Operators should provide:

- `DATABASE_URL` for their own Postgres database.
- `LEDGERISE_CREDENTIALS_KEY` for encrypting adapter credentials and AI provider keys at rest.
- Provider/API credentials for adapters and AI features.

Ledgerise must not log raw secrets, return stored secrets after initial creation, or proxy AI calls through a shared Ledgerise-owned key. In self-hosted and on-prem deployments, AI calls originate from the operator's environment using the operator's key. If managed cloud is introduced later, keys must be tenant-scoped and encrypted at rest.
