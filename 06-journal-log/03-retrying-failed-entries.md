# retrying failed entries

When a journal entry cannot be posted to your accounting system, Ledgerise retries it automatically. This page explains when that is enough, and what to do when it isn't.

---

## why entries fail

A posting failure means Ledgerise generated the journal entry and submitted it to the accounting system, but the accounting system responded with an error. Common causes:

- The accounting system API is temporarily unavailable or rate limiting.
- The adapter credentials have expired or been revoked.
- An account code referenced by the mapping rule no longer exists in the accounting system.

Ledgerise records the error response from the accounting system at each attempt. You can read it in the detail drawer's posting history.

---

## automatic retry

Failed entries are retried automatically with increasing wait times between attempts:

| Attempt | Wait before retry |
|---|---|
| 1st retry | 5 minutes |
| 2nd retry | 15 minutes |
| 3rd retry | 1 hour |
| 4th retry | 4 hours |
| 5th retry | 24 hours |

If the accounting system recovers during this window — for example, after a brief outage — the entry will post automatically on the next retry without any action from you.

---

## when automatic retries are exhausted

After 5 failed attempts, the entry is marked `retry_exhausted` and automatic retries stop. These entries appear on the Journal Log with a red badge and require manual investigation and retry.

`retry_exhausted` entries stay in the log until you resolve them. They do not age off or disappear on their own.

---

## how to investigate a failure

1. In the **Journal Log**, filter by **Failed** or **Retry Exhausted** status.
2. Click the entry to open its detail drawer.
3. Expand the **Posting History** section. Each failed attempt shows a timestamp and the error message returned by the accounting system.

Read the error message before retrying. Retrying without fixing the root cause will fail again immediately.

[SCREENSHOT: Journal entry detail drawer showing the posting history timeline with two failed attempts, the error message from the accounting system, and the retry button]

---

## common errors and fixes

**AUTH_FAILED**

Your accounting system credentials are no longer valid. The API key, client secret, or token has expired or been revoked.

Fix: go to **Settings → Adapters**, find your outbound adapter (Zoho Books or journal-csv), click **Configure**, re-enter valid credentials, and save. Then retry the failed entries.

**RATE_LIMITED**

The accounting system is throttling requests from Ledgerise. This is usually transient — the automatic retry schedule handles it in most cases by spacing retries out. If you see persistent rate-limit errors across many entries, check whether another integration is also hitting the same accounting system API concurrently.

Fix: wait for the next automatic retry, or retry manually after a short delay.

**Invalid account code**

The mapping rule references a COA account code that no longer exists in your accounting system — the account may have been renamed or deleted.

Fix:

1. Check your accounting system and confirm whether the account still exists. If it was deleted, recreate it or identify its replacement.
2. Go to **Settings → COA Reference → Import COA** to pull the latest account list into Ledgerise.
3. Go to **Mapping Rules**, find the affected rule, and update the debit or credit account to the correct code.
4. Retry the failed entries.

---

## how to retry manually

1. In the Journal Log, locate the failed or retry-exhausted entry.
2. Click **Retry** on the row. This triggers an immediate posting attempt outside the scheduled engine run.
3. The posting history in the detail drawer updates within seconds with the result.

You can retry one entry at a time from the table row, or open the detail drawer and use the **Retry** button there.

[SCREENSHOT: Journal Log filtered to retry-exhausted status showing the red badge and Retry action button on a row]

---

## entries that cannot be retried

`posted` entries cannot be retried — the accounting system already accepted them. `cancelled` and `duplicate` entries will never post by design. `unmapped` entries need a mapping rule assigned first — use **Assign Rule** on the row.

→ See [creating a rule](../05-mapping-rules/02-creating-a-rule.md) if you need to add a mapping rule for an unmapped entry
