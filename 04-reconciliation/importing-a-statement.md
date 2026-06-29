# importing a statement

A reconciliation run is triggered by importing an external statement — a CSV file from your payment provider or bank. This is the starting point for every reconciliation cycle.

**Who does this:** Finance Officers and Operations Officers when an external statement arrives.

---

## before you import

Make sure the following are in place:

- A **report source** exists (or you are ready to create one) for this counterparty and statement type.
- **Reconciliation rules** are configured for that report source — at minimum, a Reference Matching rule so the engine knows how to pair internal and external records.
- The statement file is a **CSV** with a header row and one transaction per row.

If this is your first time importing from a particular counterparty, you will create the report source and configure its rules as part of this flow.

---

## step 1 — open the import drawer

On the Reconciliation page, click **Import Statement** in the page header. A drawer opens on the right side.

---

## step 2 — select or create a report source

A report source identifies the counterparty and the type of statement. Its display name follows the format `Source Name — Report Name`, for example:

- `Paystack — Settlement Report`
- `GTBank — Collection Account Statement`
- `Flutterwave — Transaction Report`

**If the report source already exists:** select it from the dropdown. Its saved reconciliation scope (product line, biller, etc.) pre-populates automatically.

**If this is a new counterparty or a new report type:** click **Add Report Source** and fill in:

| Field | Options / notes |
|---|---|
| Source Name | The provider or bank name — e.g. `Paystack`, `GTBank` |
| Report Type | `Provider Report` or `Bank Statement` |
| Report Name | Select a preset or enter a custom name (see below) |
| Reconciliation Scope | Product line, biller, biller category, and/or transaction type — limits which internal records are compared against this statement |

**Provider report presets:** Transaction Report, Settlement Report, Switch Report, Processor Report, Aggregator Report, Fee Report, Chargeback Report, Reversal Report, Other.

**Bank statement presets:** Settlement Account Statement, Collection Account Statement, Payout Account Statement, Credit Report, Debit Report, Other.

The scope fields are optional but important: they tell Ledgerise which internal transaction records to pull for comparison. A `Paystack — Settlement Report` scoped to the `bill-payment` product line will only compare bill payment transactions — not wallet or lending transactions — against the Paystack file.

[SCREENSHOT: Import Statement drawer showing the report source selection step — a dropdown with existing report sources and an "Add Report Source" option, with Paystack — Settlement Report selected as the example]

---

## step 3 — set the statement date range

Enter the **from** and **to** dates covered by the statement. Ledgerise uses this range to determine which internal transactions to pull into the comparison pool.

---

## step 4 — upload the file

Drag and drop your CSV file or click to browse. Ledgerise reads the column headers and a sample of values.

Accepted formats: `.csv`. Maximum file size: 5 MB by default (configurable by your Admin).

---

## step 5 — map the file columns

The field mapping step tells Ledgerise which columns in your file correspond to the fields the matching engine needs:

| Canonical field | What to map it to |
|---|---|
| External reference | The column containing the processor's transaction ID or reference |
| Gross amount | The column containing the pre-fee transaction amount |
| Fee amount | The column containing the fee deducted by the processor (if present) |
| Net amount | The column containing the settled amount after fee deduction |
| Transaction date / value date | The column containing the settlement or transaction date |

You also set the date format (e.g. `DD/MM/YYYY`) and whether amounts are in major units (naira) or smallest units (kobo).

**If you are importing from a report source for the second time,** the mapping from your previous import is pre-loaded. Review it and adjust if the file format has changed.

**AI-assisted mapping:** When importing from a new report source, Ledgerise analyses your column headers and a sample of values and proposes a mapping. You confirm or adjust before proceeding.

[SCREENSHOT: Field mapping step in the import drawer showing CSV column names on the left and canonical field dropdowns on the right, with date format and amount unit selectors below]

---

## step 6 — confirm and run

Click **Confirm**. Ledgerise starts the reconciliation run in the background. The engine compares each external record against the internal transaction pool, applying your configured reconciliation rules.

For most statement sizes, the run completes within seconds to a minute. Large files (above 10,000 rows) may take longer, with matching and report generation processed by the worker in the background.

---

## after the run completes

When the run finishes, the UI navigates you to:

- The **Breaks tab** if any discrepancies were found. The break count and SLA status are visible immediately.
- The **Matches tab** if the run is fully clean — every external record was matched.

The run appears in the **Runs tab** with its match rate, break count, and status. A match rate above 95% is the healthy target for an established integration. A new counterparty or first import typically produces more breaks until your rules and field mappings are refined.

[SCREENSHOT: Breaks tab after a completed import showing a list of open breaks with their type badges and SLA status indicators — some amber, some green]

---

## reimporting a corrected statement

If a processor reissues a corrected statement for the same period, you can reimport it. The previous run is marked `superseded`, a new run is created with the corrected file, and an audit entry records who triggered the reimport and why. Confirmed matches from the superseded run are retained and flagged; resolved breaks are kept in the audit log.

---

## managing report sources

Report sources are created from the import drawer and managed in Settings. You can view, rename, edit the default reconciliation scope, and deactivate report sources from **Settings → Report Sources**.

Each report source's rules — reference matching logic, fee schedules, amount tolerances, and SLA thresholds — are configured on the **Reconciliation → Rules** tab.

→ [Reconciliation rules](reconciliation-rules.md)
