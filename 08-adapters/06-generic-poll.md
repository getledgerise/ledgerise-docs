# generic poll adapter

The `poll-adapter` fetches transactions from your source system on a schedule by calling its API. Use it when your system exposes a JSON API for querying transactions but does not send webhooks.

---

## when to use this adapter

Use the poll adapter when:

- Your payment system or internal service exposes a paginated JSON API for fetching transactions.
- The source system does not send webhooks, or webhook delivery is unreliable.
- You need Ledgerise to pull data on a schedule rather than receive it in real time.

For systems that actively push events, the webhook adapter gives you faster ingestion with lower overhead.

---

## how it works

1. On each poll cycle, Ledgerise calls your source system's API endpoint with the cursor from the previous run.
2. The adapter fetches new transactions since that cursor, normalises each one, and emits canonical records.
3. The cursor is updated to mark where this run ended, so the next run starts from the right place.
4. New transaction records appear in the Transactions page after each successful poll.

---

## configuration

Go to **Settings → Adapters → poll-adapter → Configure**.

| Field | Required | Description |
|---|---|---|
| API endpoint URL | Yes | The URL Ledgerise calls to fetch transactions (for example, `https://api.acme.com/v2/transactions`) |
| Authentication method | Yes | How to authenticate. Options: Bearer token, Basic auth, API key in header, API key in query string. |
| Credentials | Yes | The API key or token value. Stored encrypted using your `LEDGERISE_CREDENTIALS_KEY`. |
| Response field path | Yes | The dot-notation path to the array of transaction records inside the API response. For example, `data.transactions`. |
| Poll schedule | Yes | How often to poll. Enter a cron expression. Default: every 5 minutes (`*/5 * * * *`). |
| Field mapping | Yes | Maps canonical schema fields to paths in the JSON record returned by the API. |
| Date format | Yes | The format the API uses for date/time strings. |
| Amount format | Yes | Whether amounts are in full currency units or smallest units. |

---

## the poll cursor

The cursor tracks where the last successful poll ended, so each new poll picks up only new records without re-fetching records already ingested.

The cursor contains two fields:

- `last_fetched_at` — the timestamp of the last successful poll
- `last_source_id` — the ID of the last record fetched in the previous run

Ledgerise passes the cursor to your API as query parameters (for example, `?since=2026-06-01T14:00:00Z`). Configure the **cursor field mapping** to tell the adapter which API parameter to use.

If your API does not support cursor-based pagination — for example, it only supports date-range queries — the adapter falls back to using `last_fetched_at` alone. Document any known gaps in a short overlap window in your adapter README to avoid missing records during clock skew.

---

## monitoring poll runs

Go to **Settings → Adapters → poll-adapter**. The adapter row shows:

- **Last run** — the timestamp of the most recent poll
- **Healthcheck status** — whether the adapter can reach the source API

Individual poll run results are visible in the adapter detail panel — each run shows the record count, any errors, and the updated cursor value.

---

## troubleshooting

**No new records appearing after a poll**

Check the Last Run timestamp in Settings → Adapters. If it is recent, the poll ran but found no new records — this is normal if there has been no transaction activity since the last run. If the timestamp is stale, the poll may have failed — check the healthcheck status for an error message.

**Duplicate records appearing**

The engine deduplicates using `source_id` at posting time, so duplicates in the Transactions page with the same source ID are skipped when the engine runs. If you are seeing duplicates with different source IDs, check whether the API is returning the same transactions under different IDs — a known issue with some providers during pagination.

**Poll running but missing records**

Check the cursor field mapping configuration. If the API parameter name doesn't match what you configured, the API may be ignoring the cursor and returning a full record set each time, or returning from the wrong starting point. Enable verbose logging in your deployment and inspect the raw API request and response for a poll run.

**AUTH_FAILED healthcheck**

Your API credentials have expired or been revoked. Go to **Settings → Adapters → poll-adapter → Configure**, re-enter valid credentials, and save.
