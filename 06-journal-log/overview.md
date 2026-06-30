# journal log overview

The Journal Log is where you see what the journal engine produced: every double-entry journal entry generated from your transaction records, along with each entry's posting status.

---

## what the journal log shows

The page has two parts:

**Stat bar** — four numbers at the top of the page:

| Stat | What it means |
|---|---|
| Posted Today | Journal entries successfully submitted to your accounting system today |
| Failed | Entries that failed to post and are in the automatic retry queue |
| Unmapped | Entries that could not be matched to a mapping rule and were posted to the suspense account |
| Last Engine Run | The timestamp of the most recent engine cycle |

**Entries table** — one row per journal entry, with columns for Journal ID, Transaction ID, date, type, amount, product line, biller, debit account, credit account(s), status, and actions.

[SCREENSHOT: Journal Log page showing the stat bar with today's Posted, Failed, Unmapped counts and Last Engine Run timestamp, and the entries table below]

---

## how the engine run works

The journal engine runs on a configurable schedule — by default, every hour. Each run does the following in sequence:

1. Fetches all transactions with status `completed` and posting status `unposted`.
2. Deduplicates: skips any transaction whose source ID has already been posted.
3. For each transaction, finds the matching mapping rule.
4. Generates a balanced double-entry journal entry.
5. Submits the entries to the outbound adapter (Zoho Books or journal-csv) in batches.
6. Records the result for each entry: `posted`, `failed`, or `unmapped`.

You can trigger a run outside the schedule at any time by clicking **Run Engine Now** in the page header.

→ See [how Ledgerise works](../01-getting-started/how-ledgerise-works.md) for the full data flow

---

## row actions by status

| Status | What it means | Action on the row |
|---|---|---|
| `posted` (green) | Successfully submitted to your accounting system | **View** — opens the detail drawer |
| `failed` (red) | Submitted but the accounting system returned an error. Ledgerise is retrying automatically. | **Retry** — triggers an immediate retry outside the scheduled run |
| `retry_exhausted` (red) | Failed 5 times. Automatic retries have stopped. | **Retry** — one more manual attempt, with a warning in the drawer |
| `unmapped` (amber) | No matching mapping rule was found. Posted to the suspense account. | **Assign Rule** — opens Mapping Rules pre-filtered to this transaction's product line |

→ See [retrying failed entries](retrying-failed-entries.md) for how to investigate and fix failures

---

## filter bar

Use the filter bar above the entries table to narrow by date range, posting status, product line, biller, transaction type, or currency. Apply filters before exporting if you only need a subset of entries.

---

## who can use the journal log

| Role | Access |
|---|---|
| Admin | Full — view all entries, trigger engine runs, retry failed entries |
| Finance | Full — view all entries, trigger engine runs, retry failed entries |
| Auditor | Read-only — can view entries and detail drawers, but cannot trigger runs or retries |

---

## where to go next

- [Journal entries](journal-entries.md) — what each entry contains and how to read it
- [Retrying failed entries](retrying-failed-entries.md) — how to investigate and resolve posting failures
- [Exporting journal data](exporting-journal-data.md) — how to export entries for reporting or audit
