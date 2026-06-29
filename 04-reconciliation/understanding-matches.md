# understanding matches

A match record is a confirmed pair between an internal Ledgerise transaction and a corresponding record in an external statement, where all matching conditions were satisfied within your configured tolerances. Matches are informational — they represent completed verification and require no action.

---

## the matches tab

Go to **Reconciliation → Matches** to view all confirmed match records across all runs.

[SCREENSHOT: Matches tab showing a table of confirmed match pairs with columns for Source ID, External Reference, Internal Amount, External Amount, Match Confidence, Matched On, and Report Source — with a filter bar above and a match confidence badge visible on each row]

---

## columns explained

| Column | What it shows |
|---|---|
| **Source ID** | The internal `source_id` of the matched Ledgerise transaction |
| **External reference** | The reference from the counterparty statement |
| **Internal amount** | The transaction amount recorded in Ledgerise |
| **External amount** | The gross amount in the external statement |
| **Amount difference** | The difference between internal and external amounts — zero for exact matches, a small value for tolerance-approved matches |
| **Match confidence** | How the match was confirmed — see below |
| **Matched on** | The field or fields used to link the two records (e.g. `source_id`, `amount+date`) |
| **Report source** | The counterparty and statement type — e.g. `Paystack — Settlement Report` |
| **Run date** | When the reconciliation run that produced this match was completed |

---

## match confidence levels

Every match record carries a confidence level that tells you how it was confirmed:

| Confidence | What it means |
|---|---|
| `exact` | Internal and external references match exactly; amounts and currency are identical; date is within the settlement window. The strongest match type. |
| `tolerance_approved` | References match; amounts differ by a small amount within your configured tolerance (e.g. ±₦50). An audit log entry was created noting the difference. |
| `fuzzy` | References are similar but not identical — for example, a prefix difference or truncation between your system and the processor's reference format. Amounts and date are within tolerance. Fuzzy matches are surfaced for review. |
| `manual` | Your operations team manually linked the internal and external records because the engine could not find a match automatically. Requires a note. |

Fuzzy and manual matches warrant a quick review the first time you see them for a given report source — they may indicate a reference format issue that a new reconciliation rule can resolve automatically in future runs.

---

## match detail drawer

Click any row to open the match detail drawer. It shows both sides of the matched pair side by side:

**Internal record:**
- Transaction ID and source ID
- Amount, fee, and net amount
- Transaction date
- Status

**External record:**
- External reference
- Gross amount, fee charged, and net amount
- Value date
- Processor status

Below the side-by-side view:
- The match confidence level and the field(s) that confirmed the match
- The amount difference (if any) and whether it falls within your configured tolerance
- A link to the full transaction detail on the Transactions page
- An **Export Evidence Package** button — generates a timestamped PDF document for this match

[SCREENSHOT: Match detail drawer showing the internal and external records side by side with the match confidence badge, matched-on field, and the Export Evidence Package button in the drawer footer]

---

## filtering matches

Use the filter bar above the matches table to narrow the view:

- **By report source** — show matches for a specific counterparty and statement type only
- **By date range** — show matches from runs within a specific period
- **By match confidence** — filter to fuzzy or manual matches for a review pass

Filtering to `fuzzy` confidence is a useful periodic check. If the same fuzzy match pattern recurs across multiple runs — for example, your processor consistently truncates your source IDs — you can add a reference matching rule (strip/prefix rule) to catch these automatically in future.

---

## matches and the journal engine

By default, the journal engine posts transactions based on their transaction status (`completed`) regardless of reconciliation status. If your deployment is configured with a reconciliation posting gate, the engine will only post transactions that have cleared reconciliation to the specified level:

- `provider_matched_or_better` — transaction has at least a provider match
- `bank_matched_or_better` — transaction has a bank statement match
- `full_match_or_resolved` — both provider and bank are matched, or any breaks are resolved

Your posting gate is configured in Settings → System. If you are unsure whether your deployment uses a posting gate, ask your Admin.

→ See [reconciliation overview](overview.md) for the three evidence layers  
→ See [evidence packages](evidence-packages.md) to export a match record as an audit document
