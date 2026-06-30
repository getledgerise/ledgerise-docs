# poll adapter

The poll adapter calls your source system's API on a configurable schedule, fetches transactions that have arrived since the last successful run, and normalises them into canonical transaction records.

**Who configures this:** Admins setting up an integration with a source system that exposes a query API but does not push webhooks.

---

## how it works

1. You configure the adapter with your API endpoint, authentication details, field mapping, and a poll schedule.
2. At each scheduled interval, the worker calls your source system's API and requests transactions since the last successful poll.
3. The adapter normalises each response record into the canonical transaction schema.
4. New records are stored and appear on the Transactions page.
5. The adapter advances its cursor so the next poll only fetches records that arrived after this one.

If a poll run fails — for example, because your API is temporarily unreachable — the cursor does not advance. The next successful run will fetch from where the failed run left off, so no records are missed.

---

## configuration

Go to **Settings → Adapters → poll-adapter → Configure**.

### api endpoint

Enter the URL of your source system's transaction query endpoint. The adapter sends a GET request to this URL on each poll.

If your API uses query parameters to filter by date or cursor — for example, `?since=2024-01-15T10:00:00Z` — you can configure the parameter name and format. The adapter substitutes the cursor value automatically on each run.

### authentication

Select the authentication method your API uses:

| Method | What to configure |
|---|---|
| **API key (header)** | Header name and key value — e.g. `Authorization: Bearer <key>` |
| **API key (query param)** | Parameter name and key value — e.g. `?api_key=<key>` |
| **Basic auth** | Username and password |
| **No auth** | No credentials required (for internal APIs on a private network) |

The credentials you enter are encrypted and stored using your `LEDGERISE_CREDENTIALS_KEY`.

### response field path

Your API response is likely an envelope — the actual transaction records are nested inside a field rather than at the root level. Specify the JSON path to the records array.

For example, if your API returns:

```json
{
  "status": "success",
  "data": {
    "transactions": [ ... ]
  }
}
```

The response field path would be `data.transactions`.

### field mapping

Map each field in your API response to the corresponding canonical field, the same way you would for CSV import. Your API may use `txn_id` where Ledgerise expects `source_id`, or `settled_at` where Ledgerise expects `completed_at`.

→ See [csv import — step 1](04-csv-import.md#step-1--configure-column-mapping) for a worked example of field mapping.

### poll schedule

Set how often the adapter should run using a cron expression:

| How often | Cron expression |
|---|---|
| Every 15 minutes | `*/15 * * * *` |
| Every hour | `0 * * * *` |
| Every day at 6am | `0 6 * * *` |

Choose a frequency that balances how fresh you need your data against the load it places on your source system's API. Every 15 minutes is a good default for most production setups.

[SCREENSHOT: poll-adapter configuration panel showing the API endpoint URL, authentication type dropdown, response field path input, and poll schedule cron field]

---

## enabling the adapter

After saving your configuration, toggle the adapter to **Active**. Ledgerise runs a healthcheck — the adapter makes a lightweight test call to your API to confirm connectivity and authentication are working.

A green status badge on the adapter tile confirms the adapter is active and ready. A red or amber badge means the healthcheck failed; click the tile to see the error detail.

---

## monitoring polls

The adapter tile in Settings → Adapters shows:

- **Last run** — timestamp of the most recent poll attempt.
- **Last successful run** — timestamp of the most recent poll that returned results or confirmed no new records.
- **Status** — green (healthy), amber (last run had warnings), red (last run failed).

If a run fails, the error is shown in the adapter detail panel. Common causes include expired API keys, changes to the source API endpoint, or rate limiting.

---

## triggering a manual poll

You do not have to wait for the scheduled interval. To trigger a poll immediately:

1. Go to **Settings → Adapters → poll-adapter**.
2. Click **Run Now**.

This is useful during initial setup to verify your configuration is correct without waiting for the schedule, and after an API credential rotation to confirm the new credentials work.

---

## cursor and no-missed-records guarantee

The adapter tracks its position with a cursor — typically a timestamp or a pagination token — that marks the end of the last successful fetch.

- If a poll run fetches 200 records successfully, the cursor advances to the `completed_at` of the last record in that batch.
- If a poll run fails mid-fetch, the cursor stays where it was. The next successful run re-fetches from the same point.
- If your API returns a next-page token for paginated responses, the adapter follows pages until there are no more results before advancing the cursor.

This design means the poll adapter will never miss a record due to a transient API failure, and it will never fetch the same period twice once a run succeeds.

---

## what happens after a poll run

New records fetched from your source system appear on the Transactions page within a minute of the poll completing, with posting status `unposted`. They are picked up by the journal engine on its next scheduled run.

Duplicates detected by `source_id` — records that arrived via a previous poll run or a CSV import — are skipped and marked `duplicate`.
