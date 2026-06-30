# journal entries

A journal entry is the output the engine produces for one completed transaction — a balanced double-entry record that Ledgerise submits to your accounting system.

---

## what a journal entry contains

Each entry has:

- **Journal ID** — a unique identifier for this entry within Ledgerise.
- **Transaction ID** — the canonical transaction record this entry was generated from. Click it to open the full transaction record in the Transactions page.
- **Entry date** — derived from the transaction's `occurred_at` timestamp, not the date the engine ran.
- **Currency** — the transaction currency.
- **Debit and credit lines** — one or more debit lines and one or more credit lines. Account code, account name, and amount are shown for each line. The sum of all debits equals the sum of all credits exactly — Ledgerise validates this before submission.
- **Posting status** — the current state of this entry's delivery to your accounting system.
- **Accounting system reference** — once posted, the reference ID assigned by your accounting system (for example, a Zoho Books journal ID). This is what you use to locate the entry in Zoho Books.

---

## the entry detail drawer

Click any row in the Journal Log to open the detail drawer. It shows:

**Double-entry lines** — the full debit and credit breakdown with account codes, account names, and per-line amounts.

**Source transaction** — key fields from the canonical transaction record: source ID, amount, currency, product line, biller, and status. A link takes you directly to the full transaction record in the Transactions page.

**Mapping rule version applied** — the exact rule version that was active when this entry was generated, not the rule's current version. For example: "Electricity catch-all — v3, active from 12 Jan 2026." If the rule has since been edited or deactivated, a note says so. The rule version is fixed at the time of posting — it never changes for an already-generated entry.

**Posting history** — a timeline of every submission attempt for this entry: timestamp, result (posted, failed, rate limited), and the error message if the attempt failed.

**Accounting system reference ID** — populated once the entry posts successfully. Use this to locate the entry in Zoho Books or your other accounting system.

**Export Evidence Package** — generates a timestamped, tamper-evident document containing the canonical transaction record, this journal entry, the mapping rule version, the full posting history, and any reconciliation or flag records. → See [evidence packages](../04-reconciliation/07-evidence-packages.md)

[SCREENSHOT: Journal entry detail drawer showing the double-entry lines with debit and credit amounts, the source transaction section, the mapping rule version note, and the posting history timeline]

---

## double-entry validation

Every journal entry must balance: total debits equal total credits. Ledgerise validates this before submitting an entry to your accounting system. An entry that does not balance is held and flagged — it is never submitted in an invalid state.

If you see a `failed` entry with a validation-related error, check whether the mapping rule's credit account percentages still sum to 100. → See [retrying failed entries](03-retrying-failed-entries.md)

---

## how to trace an entry back to its source

The **Transaction ID** link in the detail drawer opens the canonical transaction record on the Transactions page. From there you can see the transaction's original source ID, the inbound adapter that received it, the raw source data, and its reconciliation status.

Going the other direction — finding the journal entry from a transaction — click any transaction row on the Transactions page to open the transaction detail drawer, which links to the posted journal entry.
