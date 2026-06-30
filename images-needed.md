# images needed

All screenshots for the Ledgerise documentation. Capture at **1440×900px** with realistic sample data (not lorem ipsum). Use numbered callouts where the body text references specific UI elements.

Place finished images in the `/images/` folder (gitignored — add to the docs platform separately). The placeholder `[SCREENSHOT: ...]` markers in each page indicate exactly where each image is inserted.

Where a screenshot can be reused across pages, the reuse is noted. Capture once, reference from multiple pages.

---

## 01 — getting started

**01-introduction.md** — 1 image

| # | File name | Description |
|---|---|---|
| 1 | `nav-bar-overview.png` | Ledgerise dashboard showing the top navigation bar with Transactions, Reconciliation, Mapping Rules, Journal Log, and Settings tabs |

**02-how-ledgerise-works.md** — 1 image

| # | File name | Description |
|---|---|---|
| 2 | `architecture-diagram.png` | System architecture diagram: inbound adapters (left) → reconciliation stage → journal engine → outbound adapters (right), with internal database shown below |

**04-quickstart.md** — 5 images

| # | File name | Description |
|---|---|---|
| 3 | `dashboard-sandbox-badge.png` | Dashboard after first login showing the Sandbox badge in the top navigation bar and the Transactions page as the default view — **reuse from 02-deployment/04-first-login.md** |
| 4 | `adapters-zoho-healthcheck.png` | Settings → Adapters showing the zoho-books adapter tile with a green healthcheck status badge after successful configuration |
| 5 | `mapping-rules-completed.png` | Mapping Rules page showing several completed rules with colour-coded account code chips (blue for asset, green for income) and the Add Rule button at top right — **reuse from 05-mapping-rules/01-overview.md** |
| 6 | `journal-log-first-run.png` | Journal Log after the first engine run showing a mix of green posted and amber unmapped entries, with the Run Engine Now button visible in the page header |
| 7 | `system-production-license.png` | Settings → System after production activation showing the Production License status indicator and no Sandbox badge in the top navigation bar — **reuse from 02-deployment/05-sandbox-to-production.md and 09-licensing/03-activating-your-license.md** |

---

## 02 — deployment

**01-overview.md** — 1 image

| # | File name | Description |
|---|---|---|
| 8 | `docker-compose-ps.png` | Terminal output of `docker compose ps` showing `api`, `web`, and `worker` services all in `Up` state — **reuse from 02-docker-deployment.md** |

**02-docker-deployment.md** — 2 images

| # | File name | Description |
|---|---|---|
| 8 | `docker-compose-ps.png` | Terminal showing `docker compose ps` with `api`, `web`, and `worker` all in `Up` state |
| 9 | `healthcheck-response.png` | Browser showing the `/healthcheck` JSON response with `"repository":"postgres"` and `"db":"ok"` |

**04-first-login.md** — 2 images

| # | File name | Description |
|---|---|---|
| 10 | `login-screen.png` | Ledgerise login screen showing the email and password fields, before sign-in |
| 3 | `dashboard-sandbox-badge.png` | Dashboard after first login showing the Sandbox badge in the top navigation bar — **same image as quickstart #3** |

**05-sandbox-to-production.md** — 2 images

| # | File name | Description |
|---|---|---|
| 11 | `license-key-input.png` | Settings → System showing the license key and public key input fields before activation — **reuse from 07-settings/03-system-settings.md and 09-licensing/03-activating-your-license.md** |
| 7 | `system-production-license.png` | Settings → System after activation showing Production License status with no Sandbox badge — **same image as quickstart #7** |

---

## 03 — transactions

**01-overview.md** — 1 image

| # | File name | Description |
|---|---|---|
| 12 | `transactions-page-overview.png` | Transactions page showing the stat bar at top, filter bar, and transaction table with a mix of posted, unmapped, and pending rows with colour-coded posting status badges |

**02-ingestion-methods.md** — 1 image

| # | File name | Description |
|---|---|---|
| 13 | `adapters-inbound-tiles.png` | Settings → Adapters showing the inbound adapter tiles (webhook, csv-import, poll) with their healthcheck status badges and last-run timestamps |

**03-webhook-adapter.md** — 2 images

| # | File name | Description |
|---|---|---|
| 14 | `webhook-adapter-config.png` | Settings → Adapters → webhook-adapter configuration panel showing the endpoint URL, signing secret field, and field mapping form with source fields on the left mapped to canonical fields on the right |
| 15 | `transactions-webhook-ingested.png` | Transactions page showing a newly ingested webhook transaction with source reference TEST-001, status completed, and posting status unposted |

**04-csv-import.md** — 2 images

| # | File name | Description |
|---|---|---|
| 16 | `csv-column-mapping.png` | csv-import configuration panel showing the column mapping step with source column headers on the left and canonical field dropdowns on the right, with a date format selector at the bottom |
| 17 | `csv-import-preview.png` | Import preview screen showing a summary line ("248 rows — 243 valid, 5 errors") and a list of validation errors below with row numbers and field names |

**05-poll-adapter.md** — 1 image

| # | File name | Description |
|---|---|---|
| 18 | `poll-adapter-config.png` | poll-adapter configuration panel showing the API endpoint URL, authentication type dropdown, response field path input, and poll schedule cron field |

**07-unmapped-transactions.md** — 1 image

| # | File name | Description |
|---|---|---|
| 19 | `transactions-unmapped-filter.png` | Transactions page filtered to "Unmapped only" showing amber posting status badges on each row, with an "Assign Rule" action button visible on a highlighted row |

---

## 04 — reconciliation

**01-overview.md** — 1 image

| # | File name | Description |
|---|---|---|
| 20 | `reconciliation-page-overview.png` | Reconciliation page showing the stat bar with Matched, Breaks, Pending, SLA Breached, and Last Run stats, and the four tabs: Runs, Matches, Breaks, Rules |

**02-importing-a-statement.md** — 3 images

| # | File name | Description |
|---|---|---|
| 21 | `import-drawer-source-select.png` | Import Statement drawer showing the report source selection step — dropdown with existing report sources and an "Add Report Source" option, with Paystack — Settlement Report selected |
| 22 | `import-drawer-field-mapping.png` | Field mapping step in the import drawer showing CSV column names on the left and canonical field dropdowns on the right, with date format and amount unit selectors below |
| 23 | `breaks-tab-after-import.png` | Breaks tab after a completed import showing a list of open breaks with their type badges and SLA status indicators — some amber, some green |

**03-understanding-matches.md** — 2 images

| # | File name | Description |
|---|---|---|
| 24 | `matches-tab-table.png` | Matches tab showing confirmed match pairs with columns for Source ID, External Reference, Internal Amount, External Amount, Match Confidence, Matched On, and Report Source |
| 25 | `match-detail-drawer.png` | Match detail drawer showing the internal and external records side by side with the match confidence badge, matched-on field, and the Export Evidence Package button in the drawer footer |

**04-resolving-breaks.md** — 3 images

| # | File name | Description |
|---|---|---|
| 26 | `breaks-tab-table.png` | Breaks tab showing a table of open breaks with break type badges (colour-coded), SLA status column with red/amber/green indicators, age in days, owner column, and a Resolve button on each row |
| 27 | `break-resolve-drawer-step1.png` | Break resolution drawer Step 1 showing the internal and external records side by side, the computed amount difference, age and SLA indicator, and the footer with Close, Export Evidence Package, and Resolve → buttons |
| 28 | `break-resolve-drawer-step2.png` | Break resolution drawer Step 2 showing the resolution type dropdown, resolution note textarea, and the Similar Breaks panel below with a checklist of related breaks |

**05-reconciliation-rules.md** — 4 images

| # | File name | Description |
|---|---|---|
| 29 | `recon-rules-tab-overview.png` | Reconciliation → Rules tab showing two report source sections (Paystack — Settlement Report and GTBank — Collection Account Statement) each expanded with rule categories, rule count badges, and version indicators |
| 30 | `recon-rules-fee-schedule-card.png` | Fee Schedule section in the Rules tab showing an active fee schedule card with tier summary text: "Card: 1.5% + ₦100, cap ₦2,000 · Bank transfer: ₦50 flat" and an Edit button |
| 31 | `recon-rules-add-edit-drawer.png` | Add/Edit Rule drawer showing the category dropdown set to "Fee Schedule", with the fee type, percentage, flat amount, and cap amount fields visible below |
| 32 | `recon-rules-version-history.png` | Version History section in a rule edit drawer showing a timeline: "v2.1 — Published by Kemi Adeyemi, 15 Jan 2026, 14:32" (Current badge) and "v2.0 — Published by Emeka Okafor, 02 Dec 2025, 09:15" below it |

**06-generating-reports.md** — 3 images

| # | File name | Description |
|---|---|---|
| 33 | `generate-report-drawer-date.png` | Generate Report drawer showing the four mode tabs — By Date selected — with period chips (Today, This Week, This Month, Custom Range) visible and This Month highlighted |
| 34 | `generate-report-drawer-nl.png` | Natural Language mode in the Generate Report drawer showing the text input with the AI badge and several example prompt chips below ("Breaks from last week", "Fee mismatches in January", "GTBank runs below 95%") |
| 35 | `report-viewer.png` | Report viewer showing a reconciliation report with a match rate stat grid at the top (9,847 matched of 9,872 total — 99.7%), followed by a break analysis section and a fee analysis section |

**07-evidence-packages.md** — 2 images

| # | File name | Description |
|---|---|---|
| 27 | `break-resolve-drawer-step1.png` | Break resolution drawer Step 1 footer showing three buttons: "Close" on the left, "Export Evidence Package" in the centre, and "Resolve →" on the right — **same image as 04-resolving-breaks.md #27** |
| 36 | `evidence-package-pdf.png` | Evidence package PDF in the report viewer showing Section 1 (header with document ID LR-EVP-20260615-003847 and generation timestamp) and Section 3 (internal and external records side by side with the computed amount difference) |

---

## 05 — mapping rules

**01-overview.md** — 1 image

| # | File name | Description |
|---|---|---|
| 37 | `mapping-rules-page-overview.png` | Mapping Rules page showing the stat bar (Active Rules, Inactive, Unmapped Today) and the rules table with colour-coded account code chips |

**02-creating-a-rule.md** — 3 images

| # | File name | Description |
|---|---|---|
| 38 | `add-rule-drawer-completed.png` | Add Rule drawer showing all fields completed for an electricity bill payment rule — product line "bill-payment", biller "ikeja-electric", debit account, and two credit account rows with percentages |
| 39 | `browse-coa-modal.png` | Browse COA modal opened from the debit account field, showing the full account list with colour-coded type chips |
| 40 | `split-credit-accounts.png` | Credit accounts section showing two rows with account picker and percentage inputs summing to 100 |

**04-chart-of-accounts.md** — 1 image

| # | File name | Description |
|---|---|---|
| 41 | `coa-reference-list.png` | Settings → COA Reference showing the account list with colour-coded type chips — blue for Asset, green for Income, and so on |

**05-rule-audit-trail.md** — 1 image

| # | File name | Description |
|---|---|---|
| 42 | `rule-version-history.png` | Rule detail drawer showing the Version History timeline with version numbers, user attribution, and dates |

---

## 06 — journal log

**01-overview.md** — 1 image

| # | File name | Description |
|---|---|---|
| 43 | `journal-log-overview.png` | Journal Log page showing the stat bar with today's Posted, Failed, Unmapped counts and Last Engine Run timestamp, and the entries table below |

**02-journal-entries.md** — 1 image

| # | File name | Description |
|---|---|---|
| 44 | `journal-entry-detail-drawer.png` | Journal entry detail drawer showing the double-entry lines with debit and credit amounts, the source transaction section, the mapping rule version note, and the posting history timeline |

**03-retrying-failed-entries.md** — 2 images

| # | File name | Description |
|---|---|---|
| 45 | `journal-entry-failed-drawer.png` | Journal entry detail drawer showing the posting history timeline with two failed attempts, the error message from the accounting system, and the retry button |
| 46 | `journal-log-retry-exhausted.png` | Journal Log filtered to retry-exhausted status showing the red badge and Retry action button on a row |

**04-exporting-journal-data.md** — 1 image

| # | File name | Description |
|---|---|---|
| 47 | `journal-log-export-button.png` | Journal Log with filters applied (date range and status) and the Export CSV button visible in the page header |

---

## 07 — settings

**01-adapters.md** — 1 image

| # | File name | Description |
|---|---|---|
| 48 | `settings-adapters-list.png` | Settings → Adapters showing the adapter list with healthcheck status indicators (green OK / red error), last run timestamps, and the Configure button for each adapter |

**02-users-and-roles.md** — 2 images

| # | File name | Description |
|---|---|---|
| 49 | `settings-users-list.png` | Settings → Users showing the user list with name, email, role badges, last login timestamps, and the Invite User button in the page header |
| 50 | `role-picker-modal.png` | Role picker modal showing the three role options (Admin, Finance, Auditor) with short descriptions of each |

**03-system-settings.md** — 2 images

| # | File name | Description |
|---|---|---|
| 51 | `system-settings-engine.png` | Settings → System showing the engine schedule, batch size, and suspense account code fields |
| 11 | `license-key-input.png` | Settings → System showing the License section with the license key and public key input fields before activation — **same image as 02-deployment/05-sandbox-to-production.md #11** |

**04-audit-log.md** — 1 image

| # | File name | Description |
|---|---|---|
| 52 | `audit-log-page.png` | Settings → Audit Log showing several event rows with event type, actor name, target ID, and timestamp columns, and the filter bar at the top |

---

## 08 — adapters

**05-generic-csv.md** — 1 image

| # | File name | Description |
|---|---|---|
| 16 | `csv-column-mapping.png` | CSV import flow showing the column mapping step with dropdown pickers for each canonical field against a preview of the uploaded file — **same image as 03-transactions/04-csv-import.md #16** |

**07-zoho-books.md** — 1 image

| # | File name | Description |
|---|---|---|
| 53 | `zoho-books-config-panel.png` | zoho-books adapter Configure panel showing the Client ID, Client Secret, and Organisation ID input fields, and the Authorize button |

---

## 09 — licensing

**03-activating-your-license.md** — 2 images

| # | File name | Description |
|---|---|---|
| 11 | `license-key-input.png` | Settings → System showing the License section with the license key and public key input fields before activation — **same image as 02-deployment/05-sandbox-to-production.md #11** |
| 7 | `system-production-license.png` | Settings → System after successful activation showing Production License status and the environment mode indicator — **same image as quickstart #7** |

---

## 10 — data management

**01-backups.md** — 1 image

| # | File name | Description |
|---|---|---|
| 54 | `pg-dump-terminal.png` | Terminal showing a `pg_dump` command completing successfully with the output file path and size |

---

## summary

| Section | Unique images | Reused images | Total placements |
|---|---|---|---|
| 01 getting-started | 5 | 1 | 9 |
| 02 deployment | 5 | 2 | 7 |
| 03 transactions | 8 | 0 | 8 |
| 04 reconciliation | 17 | 1 | 18 |
| 05 mapping-rules | 6 | 0 | 6 |
| 06 journal-log | 5 | 0 | 5 |
| 07 settings | 6 | 1 | 7 |
| 08 adapters | 2 | 1 | 2 |
| 09 licensing | 0 | 2 | 2 |
| 10 data-management | 1 | 0 | 1 |
| **total** | **54 unique images** | — | **65 placements** |
