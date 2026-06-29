# generating reports

A reconciliation report is the primary evidence document for a completed reconciliation run or period. It contains a run summary, match rate statistics, SLA status, break analysis by type, and a full resolution log. Every report generates both a PDF summary and a CSV data export simultaneously.

---

## what a report contains

**Summary section**
- Total internal records in scope
- Total external records in the imported statement
- Matched records count and match rate percentage
- Breaks raised by type
- Breaks resolved count
- Outstanding open breaks
- Net unreconciled exposure (sum of amount differences on open breaks)

**Match detail section**
A table of all confirmed matches: internal reference, external reference, amount, match confidence, and matched-on field.

**Break detail section**
A table of all breaks: break ID, break type, references from both sides, amounts, age, SLA status, resolution status, and resolution type if resolved.

**Fee analysis section**
Your configured fee schedule versus the fees observed in the processor report — total fees expected, total fees charged, and the variance. Useful for detecting fee schedule drift over time.

**Open breaks section**
Breaks that remain unresolved at the time the report was generated, with age and SLA status.

**Resolution log**
A chronological record of every break resolution during the report period: who resolved it, when, and with what resolution type and note.

**Audit trail**
All reconciliation events from `recon_audit_log` during the period.

---

## how to generate a report

1. On the Reconciliation page, click **Generate Report** in the page header.
2. A drawer opens. Choose a generation mode (see below).
3. Click **Generate Report →**.
4. The report viewer opens at `reports.html` and renders the scoped report.

[SCREENSHOT: Generate Report drawer showing the four mode tabs — By Date selected — with period chips (Today, This Week, This Month, Custom Range) visible and This Month highlighted]

---

## generation modes

### by date

Select a predefined period or enter a custom date range.

| Chip | Scope |
|---|---|
| Today | Current calendar day |
| This Week | Monday through today |
| This Month | First of the month through today |
| Custom Range | Enter a from and to date |

Use this mode for regular weekly or monthly reconciliation reporting.

### by run

Select one or more specific reconciliation runs from a scrollable list. Each row shows: run ID, report source, date, and match rate. You can combine runs from different report sources into a single report.

Use this mode when you need a report scoped to a specific import batch — for example, to share evidence for a particular provider's settlement period.

### SQL / Excel

A code input for technical users who need precise filtering. Toggle between SQL query syntax and Excel formula syntax. Both are interpreted server-side to filter the data set.

Use this mode when by-date and by-run scoping is not precise enough — for example, "all breaks from Paystack runs where the match rate dropped below 95%."

### natural language

Describe the report you want in plain English. An AI badge indicates this mode uses your configured AI provider key (Settings → System → AI Provider).

Example prompts:
- "All breaks from last week"
- "Fee mismatches in January"
- "GTBank runs with match rate below 95%"
- "All missing external breaks resolved as matched late this month"

Example chips below the input pre-fill common queries. The AI interprets your description and generates the appropriate scope.

[SCREENSHOT: Natural Language mode in the Generate Report drawer showing the text input with the AI badge and several example prompt chips below: "Breaks from last week", "Fee mismatches in January", "GTBank runs below 95%"]

---

## the report viewer

After clicking Generate Report, the viewer opens and renders the report as a paginated PDF-style document.

**Toolbar (fixed at top):**
- **← Back** — returns to the Reconciliation page
- **Print** — sends the report to your printer or print-to-PDF
- **Download PDF** — downloads the PDF directly

A CSV data export is always generated alongside the PDF and is also available from the toolbar. The CSV contains the match detail and break detail tables in a machine-readable format for further analysis or import into other tools.

[SCREENSHOT: Report viewer showing a reconciliation report with a match rate stat grid at the top (9,847 matched of 9,872 total — 99.7%), followed by a break analysis section and a fee analysis section]

---

## export formats

Every generated report produces both formats simultaneously. There is no format choice:

| Format | What it is |
|---|---|
| **PDF** | The human-readable audit and regulatory document — formatted, stamped with generation timestamp and operator name |
| **CSV** | The raw match and break data tables for analysis — always bundled with the PDF |

For per-transaction or per-break evidence documents, see [evidence packages](evidence-packages.md).

---

## how often to generate reports

There is no fixed cadence required by Ledgerise — generate reports on whatever schedule fits your operations and compliance obligations. Typical patterns:

- **After every reconciliation run** — for high-volume operations where each provider settlement needs to be documented immediately.
- **Weekly** — consolidating all runs from the past week into a single report for the finance team's review.
- **Monthly** — a full-period report for management accounts, audit preparation, or regulatory submission.
- **On demand** — in response to a specific audit request, regulatory enquiry, or partner bank query.

The AI natural language mode is particularly useful for on-demand requests — for example, when an auditor asks for "all disputed transactions from Q1 that were resolved as chargebacks acknowledged."
