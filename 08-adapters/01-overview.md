# adapters overview

Adapters are the only layer in Ledgerise that knows about the outside world. The engine has no built-in knowledge of Paystack, Flutterwave, Zoho Books, or any other system. Adapters are what make the integration possible — and keeping them separate from the engine is what makes the system stable.

---

## two directions

**Inbound adapters** receive raw transaction data from a source system and translate it into the canonical transaction schema that Ledgerise stores internally. Every record entering Ledgerise passes through an inbound adapter.

**Outbound adapters** receive journal entries from the engine and deliver them to a target accounting system. They translate Ledgerise's internal format into whatever the accounting system expects.

Because both directions share the same contract and the same schema, adding support for a new source or destination requires only a new adapter — the journal engine and everything between never changes.

---

## inbound adapter modes

Inbound adapters support four modes, depending on how the source system sends data:

| Mode | How it works | When to use it |
|---|---|---|
| **Webhook** | Your source system pushes individual transaction events to a URL Ledgerise exposes | Real-time event-driven integrations where your payment provider or internal system calls a URL on each transaction |
| **File import** | You upload a flat file (CSV or XLSX) exported from your source system | Batch imports, legacy systems that export periodic reports, manual one-off loads |
| **Poll** | Ledgerise calls your source system's API on a schedule and fetches new transactions | Systems that expose an API but do not send webhooks |
| **Manual entry** | An operator enters a single transaction directly through the UI | Edge cases and corrections only — not a substitute for automated ingestion |

---

## built-in adapters

Ledgerise ships with the following adapters:

**Inbound:**

| Adapter | Mode | Use case |
|---|---|---|
| `webhook-adapter` | Webhook | Generic JSON webhook integration for any system that can push events |
| `csv-import` | File import | CSV and XLSX file uploads from any source |
| `poll-adapter` | Poll | Generic JSON API polling for any system that exposes a paginated transaction endpoint |

**Outbound:**

| Adapter | Use case |
|---|---|
| `zoho-books` | Post journal entries directly to Zoho Books via the Zoho Books API |
| `journal-csv` | Export journal entries as a CSV file for manual import into any accounting system |

---

## how adapters are managed

You enable, disable, and configure adapters in **Settings → Adapters**. Each adapter displays its healthcheck status, last run time, and latency. Credentials are stored encrypted using your `LEDGERISE_CREDENTIALS_KEY`.

→ See [Settings → Adapters](../07-settings/01-adapters.md)

---

## building a custom adapter

All adapters — built-in or custom — implement the same four-method interface: `meta()`, `validate()`, `normalize()`, and `healthcheck()`. If you need to integrate a source or destination system that isn't covered by a built-in adapter, you can build and register a custom one.

→ See [adapter specification](02-adapter-spec.md) for the technical contract
→ See [building an adapter](03-building-an-adapter.md) for a step-by-step walkthrough

---

## adapter pages

- [Adapter specification](02-adapter-spec.md) — the full interface contract that all adapters must implement
- [Building an adapter](03-building-an-adapter.md) — a practical walkthrough for developers
- [Generic webhook](04-generic-webhook.md) — configuring the `webhook-adapter` for your source system
- [Generic CSV import](05-generic-csv.md) — importing transaction files with the `csv-import` adapter
- [Generic poll](06-generic-poll.md) — polling an API with the `poll-adapter`
- [Zoho Books](07-zoho-books.md) — configuring the Zoho Books outbound adapter
- [Journal CSV export](08-journal-csv-export.md) — exporting journal entries as CSV for manual import
