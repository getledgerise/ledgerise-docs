# journal CSV export adapter

The `journal-csv` adapter exports Ledgerise journal entries as a CSV file. Use it when your accounting system has no API integration with Ledgerise — you export the entries here and import them into your accounting system manually.

---

## when to use this adapter

- Your accounting system (QuickBooks, Wave, Sage, or any other) is not yet supported by a Ledgerise API adapter.
- You want a file-based audit trail of all journal entries, independent of any API connection.
- You are in a transition period — using CSV export while a direct API adapter is being configured.

If your accounting system is Zoho Books, use the [Zoho Books adapter](zoho-books.md) instead for direct posting.

---

## configuration

Go to **Settings → Adapters → journal-csv → Configure**.

| Field | Description |
|---|---|
| Date format | The date format for the `entry_date` column (for example, `YYYY-MM-DD` or `DD/MM/YYYY`). Match your accounting system's expected import format. |
| Amount display unit | Whether amounts appear in full currency units (500.00) or smallest units (50000). Most accounting systems expect full currency units. |
| Filename pattern | The naming pattern for exported files. Supports date placeholders: `{YYYY}`, `{MM}`, `{DD}`. Example: `ledgerise-journals-{YYYY}-{MM}.csv`. |
| Include transaction IDs | Whether to include a `transaction_id` column in the export. Useful for traceability, but some accounting systems reject extra columns — disable if your import template is strict. |
| Include mapping rule IDs | Whether to include a `rule_id` column. Useful for audit purposes. |

---

## what the export contains

The CSV has one row per **journal line** (not per journal entry). A journal entry with two debit lines and one credit line produces three rows.

| Column | Description |
|---|---|
| `journal_id` | Unique entry identifier in Ledgerise |
| `transaction_id` | The source transaction this entry was generated from (if enabled) |
| `entry_date` | The entry date, formatted per your configuration |
| `account_code` | The COA account code |
| `account_name` | The COA account name |
| `debit_amount` | Amount debited to this account (blank if this line is a credit) |
| `credit_amount` | Amount credited to this account (blank if this line is a debit) |
| `currency` | ISO 4217 currency code |
| `product_line` | Product line from the source transaction |
| `biller` | Biller from the source transaction |
| `rule_id` | The mapping rule version applied (if enabled) |

---

## how to export

The `journal-csv` adapter works in two ways:

**From the Journal Log** — go to **Journal Log**, apply any date range or status filters you need, and click **Export CSV**. This downloads the filtered entries as a CSV file using the format configured in this adapter.

**Automatic generation** — when the engine runs and entries are generated, the adapter can be configured to write the CSV file to a configured output path on the server. Check with your infrastructure team whether this is set up.

---

## importing into your accounting system

The export format and column names vary by accounting system. Most systems that accept journal imports expect:

- One row per journal line (debit and credit on separate rows) — this matches the journal-csv format.
- Debit and credit amounts in separate columns — this also matches.

Consult your accounting system's journal import documentation for the exact template. You may need to rename columns or reorder them before importing.

Common steps:
1. Export from Ledgerise as CSV.
2. Open the file in a spreadsheet tool.
3. Reformat or rename columns to match your accounting system's import template.
4. Import through your accounting system's journal import feature.

Keep the original export file before reformatting. It is your audit record of exactly what Ledgerise generated.
