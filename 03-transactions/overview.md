# transactions

The Transactions page is the default landing page in Ledgerise. It shows every transaction record that has been ingested by your adapters — normalised, stored, and ready for the journal engine.

---

## what the transactions page shows

Every transaction that enters Ledgerise — whether it arrived via webhook, CSV upload, or scheduled poll — appears here in a consistent format. The page lets you see what has been ingested, what has been posted, and what needs your attention.

[SCREENSHOT: Transactions page with the stat bar at top, filter bar, and transaction table showing a mix of posted, unmapped, and pending rows with colour-coded posting status badges]

---

## the stat bar

At the top of the page, a stat bar shows a real-time summary of transaction activity:

| Stat | What it shows |
|---|---|
| **Today** | Total transactions ingested today |
| **Completed** | Transactions with source status `completed` |
| **Unmapped** | Transactions posted to the suspense account — no matching rule was found |
| **Flagged** | Transactions your team has manually flagged for review |
| **Pending** | Transactions not yet finalised in the source system |

The unmapped count is the most important number to watch in the early days after go-live. A rising unmapped count means mapping rules are missing for one or more transaction types.

---

## the transaction table

Each row in the table represents one transaction record. The columns you will see most often:

| Column | What it shows |
|---|---|
| **Reference** | The source system's transaction ID (`source_id`). Clicking a row opens the detail drawer. |
| **Date** | The `occurred_at` timestamp from the source system. |
| **Amount** | Transaction amount and currency. |
| **Product line** | The product line label from the source system (e.g. `bill-payment`, `wallet`). |
| **Biller** | The biller identifier, if present (e.g. `ikeja-electric`). |
| **Status** | The transaction status from the source system. |
| **Posting status** | What Ledgerise has done with this transaction. |

### row colour states

Rows use subtle background colours to help you triage at a glance:

- **No highlight** — posted or pending. No action needed.
- **Amber** — unmapped or failed. Needs attention.
- **Grey** — cancelled, duplicate, or source-failed. Informational.

---

## posting status badges

Every transaction shows a posting status badge — a coloured label indicating what Ledgerise has done with it:

| Badge | Colour | Meaning |
|---|---|---|
| `unposted` | Grey | Waiting for the next engine run |
| `posted` | Green | Journal entry created and submitted successfully |
| `unmapped` | Amber | No matching rule found; posted to suspense account |
| `failed` | Red | Posting attempted but accounting system returned an error |
| `retry_exhausted` | Red | Failed after maximum retry attempts; manual action required |
| `duplicate` | Grey | Already posted under the same source ID; skipped |
| `cancelled` | Grey | Transaction reversed before posting; no entry created |

→ Full explanations with action guidance: [transaction statuses](transaction-statuses.md)

---

## the exceptions badge

The top navigation bar shows an **Exceptions** badge when any transactions or journal entries require your attention. The count aggregates:

- Unmapped transactions (no matching mapping rule)
- Failed posting attempts
- Retry-exhausted entries
- Open reconciliation breaks

A non-zero exceptions count means something is waiting for your action. Clicking the badge takes you to a filtered view of all open issues.

---

## filtering and searching

Use the filter bar above the table to narrow the view:

- **By posting status** — filter to unmapped only, failed only, posted only, and so on.
- **By product line** — show only transactions from a specific product line.
- **By date range** — show transactions for a specific period.
- **By source adapter** — show only transactions that came from a particular adapter.
- **Search by reference** — enter a source transaction ID to find a specific record.

Filters can be combined. The active filter summary shows how many records match the current selection.

---

## transaction detail drawer

Clicking any row opens the detail drawer. It shows:

- The full canonical transaction record — all fields, including metadata.
- The source adapter that ingested it and the source environment (`live` or `test`).
- The posting status and posting history — including any error messages from failed attempts.
- The journal entry generated for this transaction, if one exists.
- A button to generate an evidence package for audit or dispute purposes.

---

## where transactions come from

Transactions enter Ledgerise through one of three configured adapters:

- **Webhook** — your source system pushes events in real time.
- **CSV import** — you upload a batch file.
- **Poll** — Ledgerise calls your source system's API on a schedule.

→ See [ingestion methods](ingestion-methods.md) for a full comparison and setup links.
