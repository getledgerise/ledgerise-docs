# ingestion methods

There are three ways transaction data can enter Ledgerise. All three produce the same result: a normalised canonical transaction record stored in the database, visible on the Transactions page, and ready for the journal engine.

---

## choosing the right method

| Method | Best for |
|---|---|
| **Webhook** | Source systems that push transaction events in real time. Zero latency from transaction completion to Ledgerise ingestion. |
| **CSV import** | Backfills, onboarding historical data, or providers that export flat files but have no webhook or API. Also useful for one-off corrections. |
| **Poll** | Source systems that expose a query API but do not push webhooks. Ledgerise calls the API on a schedule and fetches new records. |

> **Manual entry** is also available as a fallback for exceptional one-off records — for example, an adjustment that exists nowhere in your source system. It is not a substitute for the above methods and should be used sparingly.

---

## how all three methods work the same way underneath

Regardless of which method you use, every transaction goes through the same normalisation step before it is stored. The inbound adapter translates the source data — which may use different field names, date formats, amount formats, or status labels — into the Ledgerise canonical transaction schema.

Once a transaction is in the canonical format, everything downstream — reconciliation, mapping rules, the journal engine — works identically. The engine does not know or care whether a transaction arrived via webhook or CSV.

This also means deduplication works across methods. If a transaction arrives via webhook and you later import the same period via CSV, the duplicate will be detected by `source_id` and skipped. It will never be posted twice.

→ See [transaction schema reference](../12-reference/01-transaction-schema.md) for the full canonical field list.

---

## webhook

The webhook adapter receives JSON payloads pushed by your source system or payment provider to a URL Ledgerise exposes. Each payload typically represents one transaction event.

**When to use:** Your payment engine or provider supports outbound webhooks and can be configured to push completed transactions.

**Setup:** Settings → Adapters → webhook-adapter → Configure.

**Latency:** Near real-time. Transactions appear in Ledgerise within seconds of the event.

→ Full guide: [webhook adapter](03-webhook-adapter.md)

[SCREENSHOT: Settings → Adapters showing the inbound adapter tiles (webhook, csv-import, poll) with their healthcheck status badges and last-run timestamps]

---

## csv import

The CSV import adapter accepts a flat file — typically an export from your transaction system or payment provider — and processes each row as a transaction record.

**When to use:**
- You are onboarding historical transaction data.
- Your provider exports settlement reports as CSV but has no webhook or API.
- You need to correct or backfill a batch of records.

**Setup:** Settings → Adapters → csv-import → Configure (for field mapping and format settings) and → Upload File (to import a batch).

**Latency:** Records appear as soon as the upload is processed, which typically takes a few seconds for batches under 10,000 rows.

→ Full guide: [csv import](04-csv-import.md)

---

## poll

The poll adapter calls your source system's API on a configured schedule — for example, every 15 minutes — and fetches all transactions since the last successful run.

**When to use:** Your source system exposes a query API (REST or similar) but does not push webhooks.

**Setup:** Settings → Adapters → poll-adapter → Configure. You provide the API endpoint, authentication details, and a cron schedule.

**Latency:** Up to one full poll interval. With a 15-minute schedule, a completed transaction may take up to 15 minutes to appear in Ledgerise.

→ Full guide: [poll adapter](05-poll-adapter.md)

---

## where to configure adapters

All inbound adapters are configured from **Settings → Adapters**. Each adapter tile shows:

- The adapter name and mode (webhook / file import / poll)
- The healthcheck status — green (active), amber (warning), or red (error)
- The last successful run timestamp

A red or amber status means the adapter has an issue — an expired credential, an unreachable endpoint, or a misconfigured field mapping. Click the adapter tile to see the error detail and fix it.

---

## what happens after ingestion

Once a transaction record is stored:

1. It appears on the **Transactions** page with posting status `unposted`.
2. On the next journal engine run, the engine picks it up, applies mapping rules, and generates a journal entry.
3. The journal entry is submitted to your outbound adapter (Zoho Books or journal CSV).
4. The posting status updates to `posted`, `unmapped`, or `failed` depending on the outcome.

→ See [journal log](../06-journal-log/01-overview.md) to monitor engine runs and entry outcomes.
