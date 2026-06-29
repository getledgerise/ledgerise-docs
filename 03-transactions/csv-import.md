# csv import

The CSV import adapter lets you upload a flat file — exported from your transaction system or payment provider — and import it as a batch of transaction records. Each row becomes one canonical transaction record.

**Who uses this:** Finance Officers and Admins importing batches of transactions from a file.

---

## when to use csv import

- **Historical backfills** — onboarding transaction data from before you deployed Ledgerise.
- **Provider exports** — your payment provider exports settlement reports as CSV but has no webhook or query API.
- **One-off corrections** — importing a small batch of records that need to be posted separately.
- **Development and testing** — importing sample data during initial setup to test your mapping rules and adapter configuration.

CSV import is not designed for ongoing high-volume ingestion. If your source system can push webhooks or expose an API, use the webhook or poll adapter for day-to-day operations. Use CSV import as a complement — for backfills and providers that do not support automated integration.

---

## step 1 — configure column mapping

Before importing files, you need to tell the adapter how your CSV columns correspond to canonical transaction fields. This is a one-time setup step per file format.

1. Go to **Settings → Adapters → csv-import → Configure**.
2. Upload a small sample file (5–10 rows is enough).
3. The adapter reads the column headers from the first row and presents them alongside the canonical field list.
4. Map each column to the corresponding canonical field:
   - Your `Transaction Reference` column → `source_id`
   - Your `Amount (NGN)` column → `amount`
   - Your `Transaction Date` column → `occurred_at`
   - And so on for all required fields.
5. Configure the **date format** — tell the adapter how dates are formatted in your file, e.g. `DD/MM/YYYY` or `YYYY-MM-DD`.
6. Configure the **amount format** — specify whether amounts in your file are in the smallest currency unit (kobo, cents) or in the major unit (naira, dollars). If your file uses major units, the adapter multiplies by 100 before storing.
7. Save the mapping. Ledgerise saves this column mapping for reuse so you do not need to remap on every import.

[SCREENSHOT: csv-import configuration panel showing the column mapping step with source column headers on the left and canonical field dropdowns on the right, with a date format selector at the bottom]

---

## step 2 — upload a file

Once column mapping is configured, importing a batch is straightforward.

1. Go to **Settings → Adapters → csv-import → Upload File**.
2. Drag and drop your CSV or XLSX file, or click to browse.
3. Select the column mapping profile to apply (if you have more than one source file format).
4. Click **Preview**.

---

## step 3 — review the preview

Before committing the import, Ledgerise shows a preview screen with:

- The total number of rows found in the file.
- The number of rows that passed validation.
- Any rows that failed validation, with the column and error for each.

[SCREENSHOT: Import preview screen showing a summary line (e.g. "248 rows — 243 valid, 5 errors") and a list of validation errors below with row numbers and field names]

Common validation errors include:

| Error | Likely cause |
|---|---|
| `source_id missing` | The reference column is blank on that row |
| `invalid date format` | The date in that row does not match the configured format |
| `amount not a number` | The amount column contains text or symbols |
| `unknown status value` | The status in your file uses a label the adapter does not recognise |

Fix errors in the source file and re-upload. You do not need to re-configure the column mapping.

---

## step 4 — commit the import

If the preview looks correct, click **Import**. Ledgerise processes all valid rows and stores them as canonical transaction records. Rows with validation errors are skipped.

The import runs in the background for large files. Once complete, you will see the new records on the Transactions page with posting status `unposted`.

---

## what happens to duplicates

If a row in your CSV has a `source_id` that Ledgerise has already seen — from a previous import or from a webhook — the row is skipped silently. Its posting status is set to `duplicate`. It will not be imported again and will never be posted twice.

This means it is safe to re-import a file if you are unsure whether a previous import completed. Only new records will be added; existing records will not be duplicated.

---

## file format requirements

| Requirement | Detail |
|---|---|
| File types | `.csv` and `.xlsx` |
| Maximum file size | 5 MB (default) |
| Maximum rows | 50,000 rows per import (default) |
| Encoding | UTF-8 |
| Header row | Required — first row must contain column names |

For files larger than these limits, split the file into smaller batches and import them separately. Your Admin can raise the limits by adjusting the `RECON_IMPORT_MAX_FILE_BYTES` and `RECON_IMPORT_MAX_ROWS` environment variables.

---

## after importing

Records appear on the Transactions page immediately after the import completes. They will be picked up by the journal engine on its next scheduled run, or you can trigger a manual run from the Journal Log page.

→ See [journal log overview](../06-journal-log/overview.md) to run the engine and check posting results.
