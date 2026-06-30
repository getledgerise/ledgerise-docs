# resolving breaks

A break is a discrepancy the reconciliation engine could not automatically resolve — a missing record on one side, an amount or fee mismatch, a timing difference, or a reference that could not be paired. Breaks are the exception queue for your operations team.

Every break must be reviewed and closed with a resolution type and a note. Both are required. This is the audit trail for every exception decision made by your team.

---

## the breaks tab

Go to **Reconciliation → Breaks** to see all open breaks across all runs.

Breaks are sorted by SLA status first (breached breaks appear at the top), then by age. This ensures the most urgent items are always visible without filtering.

[SCREENSHOT: Breaks tab showing a table of open breaks with break type badges (colour-coded), SLA status column with red/amber/green indicators, age in days, owner column, and a Resolve button on each row]

---

## break types

| Break type | What it means |
|---|---|
| `missing_external` | Ledgerise has an internal record for this transaction, but the counterparty statement has no corresponding entry. The processor or bank may not have recorded or settled this transaction. |
| `missing_internal` | The counterparty statement has a record, but Ledgerise has no matching internal transaction. This can indicate a transaction that was not ingested via your adapters, a chargeback, or a processor-originated entry. |
| `amount_mismatch` | Records exist on both sides but the amounts differ beyond your configured tolerance. |
| `fee_mismatch` | The gross amount matches, but the fee deducted by the processor differs from your configured fee schedule. This often indicates a fee schedule change by the processor. |
| `status_mismatch` | Your internal record shows `completed` but the processor reports `failed`, or vice versa. |
| `timing_difference` | Records exist and amounts match, but the settlement date is outside your configured settlement window. |
| `duplicate_internal` | The same reference appears more than once in your internal records within the scope of this run. |
| `duplicate_external` | The same reference appears more than once in the external statement. |
| `mapping_failure` | The bank statement references an account number not registered in your Account Mapping rules. |

---

## the SLA indicator

Every break has an SLA threshold — a number of business days within which it should be resolved. The SLA column shows:

- **Green** — within the window, no urgency
- **Amber** — approaching the deadline (one business day remaining)
- **Red (breached)** — the threshold has passed; escalation may be triggered automatically

Default SLA thresholds by break type:

| Break type | Default SLA |
|---|---|
| `missing_internal` | 1 business day |
| `status_mismatch` | 1 business day |
| `mapping_failure` | 1 business day |
| `missing_external` | 2 business days |
| `duplicate_internal` | 2 business days |
| `duplicate_external` | 2 business days |
| `amount_mismatch` | 3 business days |
| `fee_mismatch` | 3 business days |
| `timing_difference` | 5 business days |

Your Admin can adjust these thresholds in the Escalation rule configuration on the Reconciliation → Rules tab.

---

## how to resolve a break

Click **Resolve** on any break row to open the break resolution drawer. Resolution is a two-step flow.

### step 1 — review the evidence

The drawer opens on the evidence view. You see both sides of the discrepancy side by side:

**Internal record:** transaction ID, source ID, amount, fee, net, date, status  
**External record:** reference, gross amount, fee charged, net amount, value date, processor status  
**Computed difference:** the exact variance in amount and fee

Below the evidence:
- Break type and classification
- Age in days and the SLA threshold with how many days remain before breach
- Assignment — who currently owns this break, and the history of ownership
- A link to view the source transaction on the Transactions page

[SCREENSHOT: Break resolution drawer Step 1 showing the internal record and external record side by side, the computed amount difference, age and SLA indicator, and the footer with Close, Export Evidence Package, and Resolve → buttons]

Click **Resolve →** to advance to Step 2.

### step 2 — select a resolution type and write a note

| Resolution type | When to use |
|---|---|
| `matched_late` | External evidence arrived outside the settlement window but is now confirmed. The break is closed and the case marked fully reconciled. |
| `fee_schedule_updated` | The break was caused by an outdated fee schedule. The schedule has been corrected and the match now holds. |
| `write_off` | The discrepancy is below your write-off threshold and approved for closure without further action. |
| `data_error_corrected` | A field mapping or import error caused the break. The statement has been reimported and the break no longer applies. |
| `chargeback_acknowledged` | A chargeback has been received and logged. The break is closed pending chargeback accounting in the journal engine. |
| `reversal_confirmed` | A reversal has been matched on both sides and the case is closed. |
| `manual_match_approved` | You have manually linked the internal and external records. The match is approved and the case is closed. |
| `duplicate_suppressed` | A confirmed duplicate has been suppressed. The primary record is retained. |

The resolution note is a free-text field. Write enough to explain the decision to someone reading this audit trail in six months — what you found, why you made the call, and any follow-up actions taken.

Both fields are required. You cannot close a break without both.

[SCREENSHOT: Break resolution drawer Step 2 showing the resolution type dropdown with options listed, the resolution note textarea, and the Similar Breaks panel below with a checklist of related breaks]

---

## resolving similar breaks in bulk

After you select a resolution type and write a note, a **Similar Breaks** panel appears below. This panel lists other open breaks in the same run with the same break type — for example, if you have 18 `fee_mismatch` breaks all caused by the same fee schedule discrepancy.

The panel shows:
- A checkbox to select individual breaks
- A "Select all" option at the top
- For each break: Break ID, transaction reference, source, amount, and variance

The Resolve button updates dynamically: **Resolve 1 Break** when only the current break is selected, **Resolve N Breaks** when you have checked additional similar breaks.

When you confirm, every selected break is closed with the same resolution type and note. Each creates its own audit log entry. A toast confirms how many were resolved.

This bulk flow is designed for the common pattern in Nigerian payment operations: a processor changes their fee structure mid-cycle, producing dozens of identical `fee_mismatch` breaks in a single run.

---

## assigning breaks to team members

Before resolving, you can assign a break to a specific team member for investigation. Click **Assign Owner** in Step 1 of the drawer to select a user. The assignee sees the break in their queue when they filter the Breaks tab by owner.

Reassigning is also available — if investigation reveals the break belongs to a different team member, you can change the owner without resolving.

---

## escalated breaks

A break can be escalated to a senior reviewer when it cannot be resolved at the current level. Escalated breaks are flagged with an `escalated` status and assigned to a senior reviewer. They remain open until resolved.

Auto-escalation: breaks that have been open for more than a configurable number of days beyond their SLA threshold (default: 3 business days) are automatically escalated and the senior reviewer is notified.

---

## quick filter chips

Use the quick filter chips above the breaks table to narrow the view:

| Chip | Shows |
|---|---|
| SLA breached | All breaks with `sla_status: breached` |
| Unassigned | All breaks with no assigned owner |
| Amount mismatch | All `amount_mismatch` breaks |
| Missing external | All `missing_external` breaks |
| Fee mismatch | All `fee_mismatch` breaks |

You can combine chips with the full filter bar to narrow by report source, run date, or break type.

---

## after resolving

Once a break is resolved, its status moves from `open` to `resolved`. It is removed from the default view (which shows only open breaks) but remains in the full audit trail. The resolution type, note, resolved-by user, and timestamp are all recorded permanently and cannot be modified.

If the break was resolved with `fee_schedule_updated`, check whether the same correction should be made to your fee schedule rule — so future runs from the same counterparty automatically compute the correct expected net and do not raise the same break again.

→ [Reconciliation rules — fee schedule](05-reconciliation-rules.md#fee-schedule)
