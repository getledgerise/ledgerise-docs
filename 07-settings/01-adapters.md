# adapters

Settings → Adapters is where you view, configure, enable, and disable the inbound and outbound adapters registered in your Ledgerise deployment.

---

## who can access this page

Admins have full access. Finance Officers have read-only access — they can see adapter status and configuration but cannot make changes. Auditors do not have access to Settings beyond the Audit Log.

---

## the adapter list

The page shows all registered adapters, grouped into inbound (data coming in) and outbound (journal entries going out). For each adapter, you can see:

| Column | What it shows |
|---|---|
| Name | The adapter identifier (for example, `webhook-adapter`, `zoho-books`) |
| Mode | Inbound or outbound |
| Version | The adapter version bundled in this Ledgerise deployment |
| Healthcheck status | Green (OK) or red (error) — whether the adapter can currently reach its connected system |
| Last run | Timestamp of the most recent successful data fetch or journal submission |
| Latency | Round-trip time on the most recent healthcheck |
| Toggle | Enable or disable the adapter |

If a healthcheck fails, an error notice appears under the adapter row with the error code and message. Resolve the underlying issue — usually a credential problem or network connectivity — before re-enabling the adapter.

[SCREENSHOT: Settings → Adapters showing the adapter list with healthcheck status indicators (green OK / red error), last run timestamps, and the Configure button for each adapter]

---

## default adapters

Ledgerise ships with the following adapters out of the box:

**Inbound:**

| Adapter | How it receives data |
|---|---|
| `webhook-adapter` | Your source system pushes transaction events to a URL Ledgerise exposes |
| `csv-import` | You upload a flat file exported from your source system |
| `poll-adapter` | Ledgerise calls your source system's API on a schedule and fetches new transactions |

**Outbound:**

| Adapter | What it does |
|---|---|
| `zoho-books` | Posts journal entries directly to Zoho Books via the Zoho API |
| `journal-csv` | Exports journal entries as a CSV file for manual import into any accounting system |

---

## configuring an adapter

1. Find the adapter in the list and click **Configure**.
2. A drawer opens showing the credential fields that adapter requires — API keys, secrets, endpoint URLs, field mappings, and so on.
3. Enter the values and save.

Credentials are stored encrypted using your `LEDGERISE_CREDENTIALS_KEY`. If this key is not set correctly in your environment, credential storage will fail on save.

> Always use production credentials when going live. Adapter credentials entered during sandbox setup remain in place when you activate your license — double-check that they are not test or demo API keys.

---

## enabling and disabling adapters

Use the toggle on each adapter row. Disabling an adapter stops it from receiving new data or submitting new journal entries. Existing transaction records and journal entries already in Ledgerise are not affected. If you disable an inbound adapter, transactions from that source will not appear in Ledgerise until it is re-enabled.

---

## uploading a file via the CSV adapter

The `csv-import` adapter has an **Upload File** action that the other adapters do not. Click **Upload File** on the `csv-import` row to open the import flow — select your file, map the columns to the canonical schema, and confirm the import.

→ Full guide: [CSV import adapter](../08-adapters/05-generic-csv.md)

---

## adapter healthchecks

Ledgerise runs a healthcheck against each enabled adapter on startup and before each engine run. If a healthcheck fails, the adapter is marked inactive for that run and the failure is logged. You can also trigger a manual healthcheck from the Configure drawer to test a new credential without waiting for the next engine run.

→ Full details on each adapter: [adapters overview](../08-adapters/01-overview.md)
