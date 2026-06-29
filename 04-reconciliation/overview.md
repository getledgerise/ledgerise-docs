# reconciliation

Reconciliation is the verification stage of the Ledgerise core loop. Its job is to confirm that every internal transaction record is backed by evidence from an external counterparty — your payment provider and your bank — before that transaction is classified and posted to your accounting system.

Without reconciliation, the journal engine posts from internal records alone. Those records may be accurate, but they have not been verified against what the processor or bank actually reports. Reconciliation answers the question: does our record of what happened match what the counterparty recorded?

---

## where reconciliation fits

```
Transactions in  →  Reconciliation  →  Classification  →  Journal entries out
(inbound adapters)  (vs external        (mapping rules)    (outbound adapters)
                     statements)
```

Reconciliation sits between ingestion and posting. A transaction that has been reconciled and matched carries a stronger audit trail than one posted from internal records alone. For operators under regulatory scrutiny or audit, this verification layer is essential.

---

## the three evidence layers

In practice, Nigerian fintech operators verify transactions against two external sources, applied in sequence:

**Layer 1: Provider report**
The report from whoever processed, switched, aggregated, or settled the transaction — Paystack, Flutterwave, Interswitch, Baxi, and similar. This confirms the external provider acknowledged the transaction and shows the gross amount, fee deducted, and net settlement.

**Layer 2: Bank statement**
The operator's bank statement showing actual credits to their settlement account. A transaction that matches the provider report but has no corresponding bank credit has not yet resulted in cash in the operator's account.

Matching against both layers gives you a complete picture: the provider confirmed it, and the money actually arrived.

---

## key objects

**Reconciliation run** — the top-level event triggered when you import an external statement. One run compares one statement against your internal records for the matching date range and scope. Each run produces match records and break records.

**Match record** — a confirmed pair between an internal transaction and a corresponding external record, where all matching conditions were satisfied within your configured tolerances. Match records are informational; they require no action.

**Break record** — a discrepancy the engine could not automatically resolve — a missing record on one side, an amount mismatch, a fee difference, or a timing issue. Breaks are the exception queue for your operations team.

**Reconciliation case** — the persistent container for a transaction's full reconciliation history. Each internal transaction has one active case, and provider report or bank statement evidence is added to that case over time. A case moves through lifecycle statuses from `imported` through to `closed`.

**Report source** — a saved identity for a counterparty statement type, used across imports and rules. For example, `Paystack — Settlement Report` or `GTBank — Collection Account Statement`.

---

## the reconciliation page

The Reconciliation page has four tabs:

| Tab | What it shows |
|---|---|
| **Runs** | All reconciliation runs, newest first, with match rate and break count |
| **Matches** | All confirmed match records across all runs |
| **Breaks** | The exception queue — all open discrepancies requiring action |
| **Rules** | The matching engine's configuration — reference matching, fee schedules, tolerances, and SLA thresholds |

The stat bar at the top persists across all tabs:

| Stat | Meaning |
|---|---|
| **Matched** | Confirmed matches from the most recent completed run |
| **Breaks** | Open breaks across all runs — amber if any exist |
| **Pending** | Transactions imported but no external evidence received yet |
| **SLA Breached** | Breaks that have exceeded their resolution deadline — red if any exist |
| **Last Run** | Timestamp and source of the most recent completed run |

[SCREENSHOT: Reconciliation page showing the stat bar with Matched, Breaks, Pending, SLA Breached, and Last Run stats, and the four tabs: Runs, Matches, Breaks, Rules]

---

## who does this work

**Finance Officers** — the primary users of the Reconciliation page. They import statements, review breaks, assign owners, and resolve discrepancies.

**Admins** — configure report sources, reconciliation rules, fee schedules, and SLA thresholds.

**Auditors** — read-only access. Can view runs, matches, breaks, and resolution history, and can export evidence packages. Cannot resolve breaks or modify rules.

---

## reconciliation is operator-triggered, not scheduled

Reconciliation runs are initiated by your team when an external statement arrives — not by a background scheduler. Your payment provider's settlement cycle and your bank's statement frequency determine when you reconcile. Ledgerise does not assume a fixed schedule.

→ [Importing a statement](importing-a-statement.md) — how to trigger a reconciliation run  
→ [Resolving breaks](resolving-breaks.md) — how to work through the exception queue  
→ [Reconciliation rules](reconciliation-rules.md) — how to configure the matching engine for each counterparty
