# transaction statuses

Every transaction record in Ledgerise carries two independent status fields. Understanding what each one means — and how they interact — tells you exactly where a transaction is in the system and what, if anything, you need to do about it.

---

## two types of status

**Transaction status** reflects what happened to the transaction in the source system — your payment engine or provider. Ledgerise receives this status from the source; it does not set it.

**Posting status** reflects what Ledgerise has done with the transaction from the perspective of journal generation and accounting system delivery. Ledgerise sets this value as the transaction moves through the engine.

These two statuses are independent. A `completed` transaction (the payment settled) may be `unposted` (the engine has not run yet), `posted` (the entry is in your books), or `unmapped` (the engine ran but could not find a matching rule). Read both together to understand the full picture.

---

## transaction status

| Status | What it means | Does Ledgerise post a journal entry? |
|---|---|---|
| `pending` | The transaction has not yet finalised in the source system. May still complete or fail. | No. Only completed transactions are eligible for posting. |
| `completed` | The transaction settled successfully. This is the normal end state for a payment. | Yes — this is the trigger for journal posting. |
| `failed` | The transaction did not complete. The payment was never processed. | No. Failed transactions are stored for the audit trail but never posted. |
| `reversed` | A previously completed transaction has been reversed. The payment was unwound. | Yes — a mirror journal entry is generated if the original was already posted (see below). |
| `disputed` | The transaction is under dispute resolution. Posting is held until the dispute is resolved. | Held. Not posted until the dispute status changes. |

### what happens when a completed transaction is reversed

If a transaction that has already been posted (`posting_status: posted`) later arrives with status `reversed`, Ledgerise generates a **reversal journal entry** — the same accounts and amounts as the original, but with debits and credits swapped, dated to the reversal timestamp.

If the original was not yet posted when the reversal arrived, it is simply cancelled (`posting_status: cancelled`). No entry is ever created for a transaction that was reversed before posting.

---

## posting status

| Status | Colour | What it means | What to do |
|---|---|---|---|
| `unposted` | Grey | Waiting for the next engine run. The transaction is eligible but has not been processed yet. | Nothing. The engine will pick it up on the next scheduled run, or you can trigger a run manually from Journal Log. |
| `posted` | Green | A journal entry was created and successfully submitted to your accounting system. | Nothing. This is the successful end state. |
| `unmapped` | Amber | The engine ran but could not find a matching mapping rule for this transaction's product line and biller. The transaction was posted to the suspense account. | Create a mapping rule for this product line and biller, then the engine will re-evaluate on the next run. → [unmapped transactions](07-unmapped-transactions.md) |
| `failed` | Red | The engine generated a journal entry and submitted it, but the accounting system returned an error. Ledgerise is retrying automatically. | Check the error in the detail drawer → posting history. Fix the root cause (e.g., an invalid account code) then retry manually if needed. → [retrying failed entries](../06-journal-log/03-retrying-failed-entries.md) |
| `retry_exhausted` | Red | The posting was attempted five times and failed each time. Automatic retries have stopped. | Open the detail drawer, read the error, fix the root cause, and click **Retry** manually. The entry will not re-enter the automatic retry queue. |
| `duplicate` | Grey | A record with the same `source_id` has already been posted. This record was skipped. | No action needed. This is the deduplication mechanism working correctly. Investigate if you see unexpected duplicates. |
| `cancelled` | Grey | The source transaction was reversed before a journal entry was posted. No entry will ever be created for this record. | No action needed. |

---

## retry schedule for failed postings

When a posting fails, Ledgerise retries automatically with increasing wait times:

| Attempt | Wait before retry |
|---|---|
| 1st retry | 5 minutes |
| 2nd retry | 15 minutes |
| 3rd retry | 1 hour |
| 4th retry | 4 hours |
| 5th retry | 24 hours |

After the fifth failed attempt, the status moves to `retry_exhausted` and automatic retries stop. The entry remains in the Journal Log and requires manual action.

---

## reading the combination

Here is how to interpret the two statuses together for the most common combinations:

| Transaction status | Posting status | What it means |
|---|---|---|
| `completed` | `unposted` | Transaction settled, waiting for the engine to run |
| `completed` | `posted` | Transaction settled and journal entry is in the accounting system — done |
| `completed` | `unmapped` | Transaction settled but no mapping rule matched — suspense account hold, action needed |
| `completed` | `failed` | Transaction settled, entry generated, accounting system rejected it — fix and retry |
| `completed` | `retry_exhausted` | Transaction settled, all retries failed — manual intervention required |
| `failed` | `unposted` | Payment failed in source system — stored for audit trail, will never be posted |
| `reversed` | `posted` | Reversal triggered a mirror journal entry — both the original and the reversal are in the books |
| `reversed` | `cancelled` | Transaction reversed before it was posted — no entries exist for this record |
| `pending` | `unposted` | Payment not yet settled — waiting for the source system to finalise it |

---

## filtering by status

From the Transactions page, use the filter bar to show only records with a specific posting status. Common filters:

- **Unmapped only** — shows all transactions in the suspense account that need a mapping rule.
- **Failed only** — shows all transactions where posting was attempted but the accounting system returned an error.
- **Retry exhausted** — shows records that need manual retry.
- **Unposted** — shows transactions queued for the next engine run.

You can combine posting status filters with date range, product line, or adapter filters.
