# exporting journal data

There are two ways to get journal data out of Ledgerise: the Journal Log CSV export for reporting and audit, and the journal-csv outbound adapter for accounting systems without a direct API integration.

---

## export the journal log as CSV

To export a filtered slice of the Journal Log:

1. Go to **Journal Log**.
2. Apply any filters you want — date range, posting status, product line, biller, currency, or transaction type.
3. Click **Export CSV** in the page header.
4. A CSV file downloads with all entries matching your current filters.

Apply filters before exporting. Without filters, the export includes all entries in the log, which can be large for high-volume operators.

[SCREENSHOT: Journal Log with filters applied (date range and status) and the Export CSV button visible in the page header]

---

## what the export contains

The export includes all table columns plus the full entry lines for each entry:

| Column | Description |
|---|---|
| Journal ID | Unique entry identifier in Ledgerise |
| Transaction ID | The source transaction this entry was generated from |
| Entry date | From the transaction's `occurred_at` field |
| Type | Transaction type |
| Amount | Transaction amount |
| Currency | Transaction currency |
| Product line | Product line tag from the transaction |
| Biller | Biller tag from the transaction |
| Posting status | `posted`, `failed`, `unmapped`, etc. |
| Debit account | Account code and name |
| Credit account(s) | Account code, name, and percentage per credit line |
| Accounting system reference | The reference ID assigned by Zoho Books or your accounting system, if posted |
| Mapping rule version | The rule version applied when this entry was generated |

---

## common uses for the export

**Month-end reporting** — export the full `posted` slice for the month. The export gives your finance team the debit/credit breakdown by account code, ready for reconciliation against accounting system records.

**Audit preparation** — filter by date range and export the full entry list. The mapping rule version column lets auditors trace each entry back to the rule that produced it.

**Reconciling Ledgerise output against your accounting system** — compare the entries exported from Ledgerise against the journals imported into Zoho Books. The accounting system reference column is the join key between the two.

---

## the journal-csv outbound adapter

If your accounting system has no direct API integration with Ledgerise, use the **journal-csv** adapter as your outbound adapter. Instead of posting journal entries via an API, the adapter generates entries as a CSV file that you can import manually into any accounting system.

Configure it in **Settings → Adapters → journal-csv → Configure**. You can set:

- Date format
- Amount display unit
- Filename pattern
- Whether to include source transaction IDs and mapping rule IDs in the output

The journal-csv adapter follows the same engine schedule and retry logic as the Zoho Books adapter. Entries appear in the Journal Log with their posting status just as they would with any other outbound adapter.

→ Full configuration: [journal-csv adapter](../08-adapters/journal-csv-export.md)
