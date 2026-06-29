# ledgerise documentation — content plan

**Purpose:** This file is the master content plan for the Ledgerise public user documentation. It lists every page to be written, organized by folder, with a description of what each page covers and markers for where screenshots should be placed.

**Writing voice:** Clear, professional, and accommodating. Explain concepts before mechanics. Assume the reader is competent but not familiar with Ledgerise. Never condescend. Avoid walls of text — use short paragraphs, numbered steps for procedures, and tables for reference material.

**File naming:** All filenames in lowercase with hyphens. No uppercase letters.

**Screenshot markers:** `[SCREENSHOT: description]` inline in the content outline marks where a screenshot or annotated image should be placed in the final draft.

---

## table of contents

1. [01-getting-started/](#01-getting-started)
2. [02-deployment/](#02-deployment)
3. [03-transactions/](#03-transactions)
4. [04-reconciliation/](#04-reconciliation)
5. [05-mapping-rules/](#05-mapping-rules)
6. [06-journal-log/](#06-journal-log)
7. [07-settings/](#07-settings)
8. [08-adapters/](#08-adapters)
9. [09-licensing/](#09-licensing)
10. [10-data-management/](#10-data-management)
11. [11-security/](#11-security)
12. [12-reference/](#12-reference)

---

## 01-getting-started/

The entry point for all new users. Covers what Ledgerise is, how it works at a high level, and where to go next based on role.

---

### `01-getting-started/introduction.md`

**Audience:** Everyone — Finance Officers, Admins, Auditors, and Developers reading for the first time.

**Content outline:**

- What Ledgerise is: a payment operations infrastructure layer that sits between a payment system and an accounting system.
- The problem it solves: fintech operators processing thousands of daily transactions cannot automatically translate those transactions into accurate double-entry journal entries. Manual reconciliation and posting is slow, error-prone, and does not scale.
- What Ledgerise is not: it is not an accounting system, not a transaction monitoring system, and not a replacement for a human accountant. It automates the mechanical posting work.
- The core loop in plain language: transactions flow in → Ledgerise reconciles them against external counterparty statements → classifies them using mapping rules → posts double-entry journal entries to the connected accounting system.
- Who uses Ledgerise day to day: Finance Officers configure rules and monitor journals; Admins handle deployment and user access; Auditors review the trail.
- Link forward to How Ledgerise Works and the Quickstart.

`[SCREENSHOT: Ledgerise dashboard overview showing the top navigation bar with Transactions, Reconciliation, Mapping Rules, Journal Log, and Settings]`

---

### `01-getting-started/how-ledgerise-works.md`

**Audience:** Finance Officers, Admins, anyone wanting to understand the system before configuring it.

**Content outline:**

- The three-layer architecture explained in plain language:
  - **Inbound adapters** receive raw transaction data (webhook, CSV upload, API poll) and normalize it into a standard format Ledgerise understands.
  - **The journal engine** reads those normalized records, matches them against your mapping rules, and generates double-entry journal entries.
  - **Outbound adapters** take those journal entries and post them to your accounting system (e.g., Zoho Books) or export them as a CSV you can import manually.
- How the data flows step by step, with a simple diagram.
- What "canonical transaction schema" means and why it matters — every source system looks different, but Ledgerise converts everything into one consistent format before it touches your accounting logic.
- Deduplication: Ledgerise prevents the same transaction from being posted twice using a stable source ID.
- Suspense: if a transaction cannot be matched to a mapping rule, it goes to a suspense account and is flagged for manual review rather than being silently dropped or misposted.
- Test mode: transactions from a test environment are stored in Ledgerise but never posted to your accounting system.

`[SCREENSHOT: Diagram of the three-layer architecture — inbound adapters → journal engine → outbound adapters]`

---

### `01-getting-started/key-concepts.md`

**Audience:** Finance Officers and Admins encountering Ledgerise terminology for the first time.

**Content outline:**

A concise reference for the key terms used throughout this documentation. Each entry is 2–4 sentences — enough to understand the concept, with a link to the relevant section where it is used in practice.

Terms to cover:
- **Adapter** (inbound and outbound)
- **Canonical transaction schema**
- **Chart of Accounts (COA)**
- **Double-entry bookkeeping**
- **Journal entry**
- **Mapping rule**
- **Product line**
- **Biller / biller category**
- **Suspense account**
- **Reconciliation run**
- **Break** (reconciliation discrepancy)
- **Match record**
- **Evidence package**
- **Posting status** (unposted, posted, unmapped, failed, retry_exhausted)
- **Transaction status** (pending, completed, failed, reversed, disputed)
- **Source environment** (live vs test)
- **Engine run**

---

### `01-getting-started/quickstart.md`

**Audience:** Admins setting up Ledgerise for the first time.

**Content outline:**

A task-oriented guide that takes the reader from a fresh deployment to their first posted journal entry. Steps are numbered and sequential. Each step references the full guide for details.

Steps:
1. Deploy Ledgerise (Docker or VPS — link to Deployment section).
2. Sign in with the default sandbox credentials.
3. Connect your accounting system: go to Settings → Adapters → configure the Zoho Books (or journal-csv) outbound adapter.
4. Sync your Chart of Accounts: Settings → COA Reference → Sync Now.
5. Enable an inbound adapter: configure the webhook, CSV, or poll adapter for your transaction source.
6. Create your first mapping rule: Mapping Rules → Add Rule.
7. Import a test transaction batch or wait for a webhook to arrive.
8. Watch the journal engine run and confirm the first entries appear in Journal Log.
9. Before going live: invite team members (Settings → Users), configure your suspense account code, then activate your commercial license in Settings → System.

`[SCREENSHOT: First-time setup checklist — Settings > Adapters showing adapter list with healthcheck status indicators]`

`[SCREENSHOT: A completed journal entry in the Journal Log with posted status badge]`

---

## 02-deployment/

For the Admin or DevOps person responsible for installing and running Ledgerise. Technical content — commands, environment variables, and checklists.

---

### `02-deployment/overview.md`

**Audience:** Admins.

**Content outline:**

- Ledgerise is customer-managed: you run it in your own infrastructure. Ledgerise does not host your transaction data.
- Two supported paths: Docker deployment (the primary commercial path) and VPS source deployment (for internal demos or development).
- What the system needs: a server, a PostgreSQL database, a license key, and your credentials for adapters and your accounting system.
- What Ledgerise provides: a versioned Docker image, this documentation, and implementation support for commercial customers.
- Brief architecture: three services (api, web, worker) + your database.
- Link forward to docker-deployment.md and vps-deployment.md.

`[SCREENSHOT: docker compose ps output showing api, web, and worker services all running]`

---

### `02-deployment/docker-deployment.md`

**Audience:** Admins.

**Content outline:**

- Prerequisites: Docker, Docker Compose, a running PostgreSQL instance, your .env values.
- Step 1: Pull the Ledgerise image (commercial customers pull from the private registry; the command is provided in your onboarding email).
- Step 2: Configure your `.env` file — copy `.env.example` to `.env` and fill in required values. Link to environment-variables.md.
- Step 3: Run database migrations.
- Step 4: Start the services (`docker compose up -d api web worker`).
- Step 5: Verify with the health check endpoint.
- Notes on the migrate tool service and why migrations must run before the app starts.
- Notes on `VITE_API_BASE_URL` — this is compiled into the web bundle at build time, not read at runtime.

`[SCREENSHOT: Terminal showing docker compose up output with all three services starting successfully]`

`[SCREENSHOT: Browser showing the /healthcheck response with repository: postgres and db: ok]`

---

### `02-deployment/vps-deployment.md`

**Audience:** Admins deploying from source (internal demos, development).

**Content outline:**

Note at the top: this path is for source-based deployments only. Commercial customers use Docker.

- Install Node.js 20, PostgreSQL, and nginx.
- Create the database and database user.
- Clone the repository and build.
- Configure the `.env` file.
- Run migrations.
- Set up systemd service files for the API and worker.
- Configure nginx to serve the web frontend and proxy API requests.
- Add TLS with Certbot.
- Verify the deployment.

`[SCREENSHOT: nginx configuration file with the proxy pass rules highlighted]`

---

### `02-deployment/environment-variables.md`

**Audience:** Admins.

**Content outline:**

Complete reference table for all environment variables. Grouped into sections:

- **Required (core):** DATABASE_URL, AUTH_TOKEN_SECRET, LEDGERISE_CREDENTIALS_KEY, VITE_API_BASE_URL.
- **Optional (runtime behavior):** API_PORT, WEB_PORT, RUN_ENGINE_ON_START, RUN_GENERIC_POLL_ON_START, RUN_GENERIC_POLL_SCHEDULE, RUN_RECONCILIATION_QUEUE_WORKER.
- **Conditional (adapters and AI):** ZOHO_CLIENT_ID, ZOHO_CLIENT_SECRET, ZOHO_ORGANIZATION_ID, AI_PROVIDER, AI_API_KEY.
- **Reconciliation import limits:** RECON_IMPORT_MAX_FILE_BYTES, RECON_IMPORT_MAX_ROWS, RECON_IMPORT_MAX_REQUEST_BYTES.

For each variable: name, required/optional, default, and a plain-English description of what it controls.

Note on secrets: AUTH_TOKEN_SECRET and LEDGERISE_CREDENTIALS_KEY are security-sensitive. Instructions for generating each (e.g., `openssl rand -hex 32` for the credentials key).

---

### `02-deployment/first-login.md`

**Audience:** Admins after a fresh deployment.

**Content outline:**

- Fresh deployments start in sandbox mode.
- Default credentials: `admin@ledgerise.dev` / `password`.
- What the Sandbox badge means: data you import here is demo data. It does not affect your accounting system.
- Walk through the first login: open the dashboard URL, sign in, confirm the Sandbox badge is visible.
- What to do in sandbox mode before going live: configure adapters, sync COA, set up mapping rules, import test transactions, and invite your team.
- How to reset sandbox data when you are ready for production (Settings → System → Reset sandbox data).

`[SCREENSHOT: Ledgerise login screen with sandbox badge visible in the top chrome after sign-in]`

---

### `02-deployment/sandbox-to-production.md`

**Audience:** Admins completing initial setup before go-live.

**Content outline:**

A step-by-step checklist. Each item explains not just what to do but why it matters.

Pre-activation checklist:
- Confirm environment variables are set for the real deployment.
- Invite named admin and finance users. Do not go live with only the default admin account.
- Configure all adapters with production credentials.
- Sync your Chart of Accounts.
- Build out your mapping rules and test them against sample transactions.
- Configure report sources and reconciliation rules for your counterparties.
- Reset sandbox data: Settings → System → Reset sandbox data. This clears demo transactions, journals, reconciliation runs, and related operational records.

Activation:
- Enter your commercial license key and public key in Settings → System.
- Confirm the Sandbox badge disappears and the deployment reads Production License.
- Confirm `/healthcheck` returns `environment_mode: "production"`.

Post-activation:
- Start real data imports only after production activation is confirmed.
- Monitor the Journal Log closely for the first 48 hours.

`[SCREENSHOT: Settings > System showing the license key input fields and the production activation confirmation screen]`

`[SCREENSHOT: /healthcheck response after production activation showing environment_mode: production]`

---

### `02-deployment/upgrading.md`

**Audience:** Admins.

**Content outline:**

- Ledgerise publishes versioned image releases. You will be notified by email with a changelog and migration notes.
- How to upgrade (Docker): pull the new image, run migrations, restart services.
- How to upgrade (VPS): pull the latest source, install dependencies, build, run migrations, restart services.
- Database migrations are always additive in minor and patch releases — no columns or tables are dropped. Upgrades are safe to run without a maintenance window.
- Version support window: 12 months per major version. After the window closes, upgrade to continue receiving security patches.
- If something goes wrong: restore your last database backup and roll back the image to the previous version tag.

`[SCREENSHOT: docker compose pull output showing the new image being pulled, followed by the migrate run]`

---

## 03-transactions/

Covers how transaction data enters Ledgerise and how to work with the Transactions page.

---

### `03-transactions/overview.md`

**Audience:** Finance Officers, Admins.

**Content outline:**

- The Transactions page is the default landing page in Ledgerise.
- What it shows: every inbound normalized transaction record produced by your adapters.
- Key stat bar: total transactions today, completed, unmapped (amber), flagged (amber), pending.
- How to read the transaction table: columns, row color states, and what each posting status badge means.
- The exceptions badge in the top nav: what it aggregates and how to use it to triage quickly.
- Link forward to specific ingestion method guides.

`[SCREENSHOT: Transactions page with the stat bar at top, filter bar, and table with a mix of posted, unmapped, and flagged rows]`

---

### `03-transactions/ingestion-methods.md`

**Audience:** Admins and Finance Officers who manage data sources.

**Content outline:**

- Overview of the four ways transactions can enter Ledgerise: webhook, CSV file import, scheduled API poll, and manual entry.
- When to use each: webhook for real-time push from payment providers; CSV for onboarding, backfills, and providers without an API or webhook; poll for providers with an API but no webhook; manual entry for exceptional one-off records only.
- All four methods normalize data to the same canonical format before storing — so everything downstream (reconciliation, mapping rules, journal engine) sees the same structure regardless of source.
- Where to configure each: Settings → Adapters.
- Link to individual adapter guides.

`[SCREENSHOT: Settings > Adapters showing inbound adapters list with their modes (webhook, file import, poll) and healthcheck status]`

---

### `03-transactions/webhook-adapter.md`

**Audience:** Admins and developers integrating a source system via webhook.

**Content outline:**

- What the webhook adapter does: receives JSON payloads pushed by your transaction system or payment provider to a URL Ledgerise exposes.
- How to configure it: in Settings → Adapters → webhook-adapter → Configure. Fields: webhook endpoint path, shared signing secret, field mapping (map source JSON fields to Ledgerise canonical fields).
- Field mapping explained: if your source payload uses `tx_amount` instead of `amount`, you tell the adapter how to find it.
- Webhook security: the adapter verifies the signature on every incoming payload. Payloads that fail verification are rejected with a 401.
- Testing: how to send a test payload and confirm the transaction appears in the Transactions page.
- What happens if the webhook fails: Ledgerise acknowledges receipt within 200ms. Normalization happens asynchronously. Adapter errors appear in the Transactions page and in adapter logs.

`[SCREENSHOT: Settings > Adapters > webhook-adapter configuration panel showing the endpoint URL and field mapping form]`

`[SCREENSHOT: Transactions page showing a newly ingested webhook transaction with status completed and posting status unposted]`

---

### `03-transactions/csv-import.md`

**Audience:** Finance Officers and Admins importing a batch of transactions from a file.

**Content outline:**

- When to use CSV import: backfills, one-off batches, or when your source system exports transaction data as CSV/XLSX.
- How to import: Settings → Adapters → csv-import → Upload File. Walk through the file upload flow step by step.
- Column mapping: map each column in your source file to the corresponding canonical field. Ledgerise saves your mapping for reuse.
- Configuring date and amount formats: how to tell the adapter how dates are formatted in your file (e.g., `DD/MM/YYYY`) and whether amounts are in full currency units (the adapter converts to smallest unit before storing).
- Validation: before committing records, Ledgerise shows a preview with any validation errors. Fix errors in the source file and re-upload if needed.
- After import: records appear in the Transactions page with posting status `unposted`. The engine picks them up on the next run.

`[SCREENSHOT: CSV import drawer showing the column mapping step with source columns on the left and canonical field dropdowns on the right]`

`[SCREENSHOT: Import preview screen showing a row count summary and any validation errors before committing]`

---

### `03-transactions/poll-adapter.md`

**Audience:** Admins configuring scheduled API polling from a source system.

**Content outline:**

- What the poll adapter does: calls your configured API endpoint on a schedule, fetches transactions since the last successful run, and normalizes them.
- How to configure: Settings → Adapters → poll-adapter → Configure. Fields: API endpoint URL, authentication method (header or token), response field path for the records array, poll schedule (cron expression), and column-to-canonical field mapping.
- The poll cursor: the adapter tracks where it left off using a cursor. If a poll run fails, the cursor does not advance — so no records are missed.
- Monitoring: check the adapter's Last Run time and healthcheck status in Settings → Adapters. A failed poll run will show an error message.

`[SCREENSHOT: poll-adapter configuration panel showing API endpoint, auth header fields, and poll schedule input]`

---

### `03-transactions/transaction-statuses.md`

**Audience:** Finance Officers.

**Content outline:**

Two types of status on every transaction record:

**Transaction status** (what happened to the transaction in the source system):
- `pending` — not yet finalized.
- `completed` — settled. Eligible for journal posting.
- `failed` — did not complete. Stored but never posted.
- `reversed` — previously completed, now reversed. Triggers a mirror journal entry if the original was already posted.
- `disputed` — flagged for dispute resolution. Posting is held.

**Posting status** (what Ledgerise has done with the transaction):
- `unposted` — awaiting the next engine run.
- `posted` — journal entry created and successfully submitted to the accounting system.
- `unmapped` — no matching mapping rule was found. Posted to suspense account.
- `failed` — posting was attempted but the accounting system returned an error.
- `retry_exhausted` — failed and retried the maximum number of times. Requires manual intervention.
- `duplicate` — a record with the same source ID was already posted. Skipped.
- `cancelled` — the original transaction was reversed before it was posted. No entry will be created.

Table showing the combinations that result in action vs. no action, with links to the relevant workflow.

---

### `03-transactions/unmapped-transactions.md`

**Audience:** Finance Officers.

**Content outline:**

- What an unmapped transaction is: the journal engine could not find a mapping rule that matched this transaction's product line, biller, and transaction type. It was posted to the suspense account.
- How to find them: Transactions page → click the "Unmapped only" quick filter, or check the stat bar for the unmapped count.
- How to resolve: click "Assign Rule" on the unmapped row. This opens the Mapping Rules page pre-filtered to the relevant product line so you can create or find the right rule.
- What happens next: after a matching rule is created, the next engine run will re-evaluate and post the transaction to the correct accounts.
- How many unmapped transactions is normal: below 2% of daily volume after initial setup is the target. Higher rates indicate missing mapping rules.

`[SCREENSHOT: Transactions page filtered to "Unmapped only" showing amber posting status badges and the "Assign Rule" action button]`

---

## 04-reconciliation/

Covers how to verify internal transaction records against external counterparty statements and how to manage discrepancies.

---

### `04-reconciliation/overview.md`

**Audience:** Finance Officers, Admins.

**Content outline:**

- What reconciliation does: verifies that every internal transaction record is backed by evidence from an external counterparty source (your payment provider and your bank).
- Where it fits in the core loop: between transaction ingestion and journal posting. A reconciled and matched transaction has a stronger audit trail than one that was simply posted from internal records alone.
- The three evidence layers: internal records → provider report → bank statement.
- Key objects: reconciliation run, match record, break record, reconciliation case.
- The Reconciliation page layout: stat bar, four tabs (Runs, Matches, Breaks, Rules).
- Who does this work: Finance Officers, with Admins managing configuration.

`[SCREENSHOT: Reconciliation page showing the stat bar (Matched, Breaks, Missing External, Last Run) and the four tabs]`

---

### `04-reconciliation/importing-a-statement.md`

**Audience:** Finance Officers.

**Content outline:**

- When to run a reconciliation: when you receive an external statement from your payment provider or bank. This is not a scheduled background job — you trigger it by importing the file.
- Step by step:
  1. Reconciliation → Import Statement.
  2. Select an existing report source or create a new one. A report source identifies the counterparty and the type of statement (e.g., "Paystack — Settlement Report", "GTBank — Collection Account Statement").
  3. Set the reconciliation scope: choose the product line, biller, biller category, and transaction type that this statement covers. This tells Ledgerise which internal records to compare against.
  4. Upload the statement file (CSV).
  5. Review the field mapping. Map columns in the file to the fields Ledgerise uses for matching (amount, reference, date, status).
  6. Confirm import. Ledgerise runs the match and lands on the Breaks tab if any discrepancies are found.
- How report sources work: the name you assign to a source (`Source Name — Report Name`) is used consistently across all your reconciliation runs and rules.
- Managing report sources: Settings → (link to settings/system-settings.md).

`[SCREENSHOT: Import Statement drawer showing the report source selection step with "Paystack — Settlement Report" as an example]`

`[SCREENSHOT: Field mapping step showing CSV columns on the left mapped to Ledgerise fields on the right]`

`[SCREENSHOT: Breaks tab after a successful import showing breaks with amber SLA indicators]`

---

### `04-reconciliation/understanding-matches.md`

**Audience:** Finance Officers.

**Content outline:**

- What a match record is: a confirmed pair between an internal Ledgerise transaction and a corresponding record in the external statement, where all matching conditions were satisfied within configured tolerances.
- How to view matches: Reconciliation → Matches tab.
- Columns explained: source ID, internal transaction date, internal amount, external amount, matched on (which field was used for matching), match confidence, report source.
- Match detail drawer: opens on row click. Shows both sides of the matched pair side by side, the match key used, and a link to export an evidence package.
- Matches are informational: they require no action. They represent completed verification.
- Filtering: filter by report source and date range to find matches for a specific counterparty or period.

`[SCREENSHOT: Matches tab showing matched pairs with the match confidence column and report source filter active]`

`[SCREENSHOT: Match detail drawer showing both sides of a matched pair side by side]`

---

### `04-reconciliation/resolving-breaks.md`

**Audience:** Finance Officers.

**Content outline:**

- What a break is: a discrepancy that could not be automatically resolved — a missing record on one side, an amount mismatch, a timing difference, or a reference mismatch.
- Break types and what they mean:
  - **Missing internal** — the counterparty has a record but Ledgerise does not.
  - **Missing external** — Ledgerise has a record but the counterparty statement does not.
  - **Amount mismatch** — both records exist but the amounts differ.
  - **Timing mismatch** — both records exist but the dates differ beyond the configured tolerance.
- The SLA indicator: breaks age from the moment they are created. The SLA column shows green (within window), amber (approaching breach), or red (breached).
- How to resolve a break (two-step drawer):
  1. Step 1: review the break detail — both sides of the evidence, break type, age, and SLA status.
  2. Step 2: select a resolution type (matched late / fee schedule updated / write-off / data error corrected / chargeback acknowledged / reversal confirmed / manual match approved / duplicate suppressed) and enter a resolution note. Both fields are required.
- Bulk resolution: if similar breaks exist (same discrepancy type and pattern), the Similar Breaks panel appears in Step 2. Select any you want to resolve with the same resolution type and note. The button updates to "Resolve N Breaks."
- A break cannot be closed without a resolution type and note. This is the audit trail for every exception decision.

`[SCREENSHOT: Breaks tab showing break types, SLA status column with red/amber/green indicators, and the Resolve button]`

`[SCREENSHOT: Break resolution drawer Step 2 showing the resolution type dropdown, note field, and the Similar Breaks panel with a checklist]`

---

### `04-reconciliation/reconciliation-rules.md`

**Audience:** Finance Officers and Admins.

**Content outline:**

- What reconciliation rules are: the configuration that tells the matching engine how to evaluate records. They are separate from the journal entry mapping rules on the Mapping Rules page.
- Where to find them: Reconciliation → Rules tab.
- Rule categories and what each controls:
  - **Reference Matching** — defines how transaction references are extracted and compared (regex patterns, exact match, fuzzy match).
  - **Account Mapping** — maps bank account numbers to GL account codes or processor identities.
  - **Amount Tolerance** — acceptable variance ranges (e.g., allow ±₦50 on transactions under ₦5,000).
  - **Fee Schedule** — per-processor fee formulas (flat fee, percentage, tiered, capped).
  - **Timing** — settlement window definitions and grace periods.
  - **Status Mapping** — maps processor-specific status codes to Ledgerise canonical statuses.
  - **Direction Check** — validates inbound/outbound and debit/credit semantics.
  - **Duplicate Detection** — time window and field combination for flagging potential duplicates.
  - **Escalation** — SLA thresholds, notification targets, and priority levels for breaks that breach their window.
- How rules are organized: grouped by report source, then by category.
- Rule versioning: each rule has a version history. When you publish an update, a new version is created. Reconciliation runs always show which rule version was active at the time.

**Creating and editing rules:**
1. Click "+ Add" on the relevant category section, or "Edit" on an existing rule card.
2. Fill in name, description, report source, category, and category-specific fields.
3. Click "Save Draft" to save without activating (useful for reviewing before publishing).
4. Click "Publish" to create a new version and activate the rule.

`[SCREENSHOT: Reconciliation > Rules tab showing rules grouped by report source with category badges and version badges (e.g., v2.1)]`

`[SCREENSHOT: Add/Edit Rule drawer showing the category dropdown and category-specific fields for Fee Schedule]`

`[SCREENSHOT: Version History section in a rule drawer showing a timeline of published versions]`

---

### `04-reconciliation/generating-reports.md`

**Audience:** Finance Officers.

**Content outline:**

- What a reconciliation report contains: run summary, match rate statistics, SLA status, and break analysis by type. Every report generates both a PDF summary and a CSV data export — no format choice required.
- How to generate a report:
  1. Reconciliation → Generate Report (button in the page header).
  2. Choose a generation mode (see below).
  3. Click "Generate Report."
  4. The report opens in the Report Viewer (reports page).
- Generation modes:
  - **By Date** — select a period chip (Today / This Week / This Month) or enter a custom date range.
  - **By Run** — select one or more reconciliation runs from a list.
  - **SQL / Excel** — write a query directly (for technical users who need precise filtering).
  - **Natural Language** — describe the report you want in plain English. Example: "All breaks from last week" or "Fee mismatches in January." Powered by AI.
- Viewing the report: the report viewer has a fixed toolbar with Back, Print, and Download PDF.
- Downloading the CSV: also available from the toolbar alongside the PDF.

`[SCREENSHOT: Generate Report drawer showing the four mode tabs — By Date selected with the period chips visible]`

`[SCREENSHOT: Natural Language mode showing the text input, AI badge, and example chips below]`

`[SCREENSHOT: Report viewer showing a reconciliation report with match rate stats and break analysis sections]`

---

### `04-reconciliation/evidence-packages.md`

**Audience:** Finance Officers, Admins, Auditors.

**Content outline:**

- What an evidence package is: a single timestamped document capturing the full record for one transaction or break, suitable for auditors, partner banks, or regulatory requests.
- What it contains: canonical transaction record, reconciliation result (match or break + resolution), mapping rule version active at time of posting, journal entry with debit/credit lines, full posting history, flag record and resolution if applicable, and a record of all user actions on this transaction.
- Where to generate one: the "Export Evidence Package" button appears in the detail drawer for a transaction (Transactions page), a match record (Reconciliation → Matches), or a break record (Reconciliation → Breaks). It also appears in the Journal Log detail drawer.
- What the output looks like: a PDF with eight sections — header with timestamps and document ID, transaction summary, match/break result, reconciliation run metadata, amount comparison table, status timeline, notes and evidence, and a footer with a digital signature.
- Typical uses: regulatory submissions, partner bank dispute resolution, internal audit requests.

`[SCREENSHOT: Transaction detail drawer showing the "Export Evidence Package" button in the footer]`

`[SCREENSHOT: Evidence package PDF showing the header section with document ID and timestamp]`

---

## 05-mapping-rules/

Covers how Finance Officers configure which accounting entries are generated for each type of transaction.

---

### `05-mapping-rules/overview.md`

**Audience:** Finance Officers, Admins.

**Content outline:**

- What mapping rules are: configuration records that tell the journal engine which COA accounts to debit and credit for a given combination of product line, biller, and transaction type.
- Why they exist: Ledgerise does not know your accounting structure. Your Finance team knows which revenue account receives airtime payments vs. electricity payments, for example. Mapping rules are how you encode that knowledge.
- No code required: Finance users create, edit, and deactivate rules entirely through the UI.
- The Mapping Rules page layout: stat bar (active rules, inactive rules, unmapped today), filter bar, rules table.
- Account code color indicators: blue = Asset, amber = Liability, green = Income, red = Expense, purple = Suspense. These make it easy to read a rule at a glance without memorizing account codes.

`[SCREENSHOT: Mapping Rules page showing the stat bar and rules table with color-coded account code chips]`

---

### `05-mapping-rules/creating-a-rule.md`

**Audience:** Finance Officers.

**Content outline:**

- How to open the Add Rule drawer: Mapping Rules → Add Rule button. The drawer opens on the right. You can see the existing rules table while creating the new one.
- Fields:
  - **Product line** (required): the operator's product line (e.g., `bill-payment`, `wallet`, `agency`). This is the primary matching key.
  - **Biller** (optional): the exact biller identifier (e.g., `ikeja-electric`). When present, this is the highest-priority match.
  - **Biller category** (optional): the category of billers (e.g., `electricity`). Used when no exact biller match exists.
  - **Transaction type filter** (optional): filter by one or more transaction types from the standard taxonomy (multi-select). When empty, the rule applies to all transaction types on the matched product line.
  - **Debit account**: searchable COA picker. Click "Browse COA" to view the full chart.
  - **Credit accounts**: one or more accounts, each with a percentage. Percentages must sum to 100. Use split credit accounts when revenue needs to be divided (e.g., 80% to revenue, 20% to a partner account).
  - **Description** (optional): a plain-English note about what this rule covers.
  - **Status**: active or inactive. Inactive rules are never evaluated by the engine.
- Save the rule. The engine uses it on the next run.
- When to create a rule for a new biller: if the "Unmapped today" stat on the Mapping Rules page is non-zero, check which product line and biller is missing a rule.

`[SCREENSHOT: Add Rule drawer showing all fields with a completed example for an electricity bill payment rule]`

`[SCREENSHOT: COA browser modal opened from the debit account field, showing accounts with color type chips]`

`[SCREENSHOT: Credit accounts section showing two rows with account picker and percentage inputs summing to 100]`

---

### `05-mapping-rules/rule-resolution-order.md`

**Audience:** Finance Officers.

**Content outline:**

- The engine evaluates rules in priority order. Understanding this order helps you write rules that work as expected.

Priority order (highest to lowest):
1. **Product line + biller (exact match)** — most specific. Use for a single biller that needs different accounting than the category default.
2. **Product line + biller category** — use for a group of billers in the same category that share the same accounts.
3. **Product line only (catch-all)** — use as a default for all transactions on a given product line that don't match a more specific rule.
4. **No match → suspense** — if no rule matches, the transaction is posted to the suspense account and flagged as unmapped.

- The engine stops at the first match. Lower-priority rules are never evaluated for a transaction that already matched.
- Practical example: you might have a catch-all rule for `bill-payment` that credits your general revenue account, and a specific rule for `bill-payment` + `ikeja-electric` that credits a dedicated electricity revenue account.
- How to see which rule was applied to a specific transaction: Journal Log → click the entry → detail drawer shows the rule version that was applied.

`[SCREENSHOT: A simple diagram or table illustrating the rule resolution priority order with arrows]`

---

### `05-mapping-rules/chart-of-accounts.md`

**Audience:** Finance Officers and Admins.

**Content outline:**

- Ledgerise does not own your Chart of Accounts. It is imported from your connected accounting system (Zoho Books, or QuickBooks when supported) and is read-only within Ledgerise.
- Where to view it: Settings → COA Reference.
- How to sync: Settings → COA Reference → Sync Now. This pulls the latest account list from your accounting system.
- What the COA Reference page shows: account code, account name, account type (with color chip), and currency.
- How to use the COA in mapping rules: the debit and credit account pickers in the Add/Edit Rule drawer are searchable. Type the account name or code to find the account. Click "Browse COA" to see the full list.
- If an account is missing: add it in Zoho Books (or your accounting system) first, then sync again in Ledgerise.

`[SCREENSHOT: Settings > COA Reference showing the account list with color-coded type chips (blue for Asset, green for Income, etc.)]`

---

### `05-mapping-rules/rule-audit-trail.md`

**Audience:** Finance Officers, Admins, Auditors.

**Content outline:**

- Every change to a mapping rule — creation, activation, deactivation, edit — is recorded automatically in an immutable audit trail.
- How to view it: Mapping Rules → click the rule row → detail drawer → Version History / Audit Trail section.
- What it shows: every version of the rule with version number, effective-from date, changed by (user), and before/after state of the rule.
- Why this matters: when an auditor asks why a particular journal entry debited one account and not another, you can show which rule version was active at the time that entry was posted.
- Journal entries also record which rule version was applied (Journal Log → entry detail drawer → "Mapping rule version applied").
- The audit trail is append-only. It cannot be edited or deleted through the UI or API.

`[SCREENSHOT: Rule detail drawer showing the Version History timeline with version numbers, user attribution, and dates]`

---

## 06-journal-log/

Covers how to monitor and manage the double-entry journal entries generated by the engine.

---

### `06-journal-log/overview.md`

**Audience:** Finance Officers, Admins, Auditors.

**Content outline:**

- The Journal Log is where you see the output of the journal engine: every double-entry journal entry generated from your transaction records, and their posting status.
- Stat bar: entries posted today, entries failed, entries unmapped, and the last engine run timestamp.
- How the engine run works: the engine runs on a schedule (configured in Settings → System). Each run picks up all completed, unposted transactions, applies mapping rules, generates journal entries, and attempts to post them to your accounting system.
- Journal Log layout: filter bar, entries table. Columns: Journal ID, Transaction ID, Date, Type, Amount, Product Line, Biller, Debit Account, Credit Account(s), Status, Actions.
- Row status and actions: posted (view), failed (retry), retry_exhausted (retry with warning), unmapped (assign rule).

`[SCREENSHOT: Journal Log page with the stat bar showing today's posted and failed counts, and the entries table below]`

---

### `06-journal-log/journal-entries.md`

**Audience:** Finance Officers, Auditors.

**Content outline:**

- What a journal entry contains: Journal ID, the source Transaction ID it was generated from, entry date (from the transaction's occurred_at), currency, the debit and credit lines (account code + amount), and posting status.
- Double-entry: every entry has at least one debit line and at least one credit line. The sum of all debits must equal the sum of all credits. Ledgerise validates this before submitting to the accounting system.
- The entry detail drawer: click any row to open it. Shows:
  - Full journal entry lines with debit/credit amounts.
  - The source transaction record (key fields + link to Transactions page).
  - The mapping rule version applied — specifically the version that was active when this entry was generated, not the current rule. Ledgerise notes if the rule has since been updated.
  - Full posting history timeline (each attempt, result, timestamp, accounting system reference).
  - The accounting system reference ID (e.g., Zoho Books journal ID).
  - "Export Evidence Package" button.
- How to trace an entry back to its source: the transaction link in the drawer opens the full canonical record in the Transactions page.

`[SCREENSHOT: Journal entry detail drawer showing the double-entry lines, mapping rule version note, and posting history timeline]`

---

### `06-journal-log/retrying-failed-entries.md`

**Audience:** Finance Officers.

**Content outline:**

- Why entries fail: the accounting system API may be temporarily unavailable, return a rate limit error, or reject an entry for a configuration reason.
- Automatic retry: failed entries are retried automatically with exponential backoff (5 minutes → 15 minutes → 1 hour → 4 hours → 24 hours). You do not need to do anything during the automatic retry cycle.
- When automatic retries are exhausted: after 5 failed attempts, the entry is marked `retry_exhausted` and requires manual action.
- How to retry manually: Journal Log → filter by status "Failed" or "Retry Exhausted" → click "Retry" on the row. This triggers an immediate retry outside the scheduled engine run.
- Investigating a failed entry: open the detail drawer and read the posting history. The error message from the accounting system is recorded at each attempt.
- Common causes and fixes:
  - **AUTH_FAILED**: re-check and re-enter your accounting system credentials in Settings → Adapters.
  - **RATE_LIMITED**: the accounting system is rate limiting. Wait and retry. Automatic retry handles this in most cases.
  - **Invalid account code**: the COA account referenced by the mapping rule no longer exists in your accounting system. Sync your COA and check the rule.

`[SCREENSHOT: Journal Log filtered to "Failed" status showing the red failed badge and Retry action button]`

`[SCREENSHOT: Journal entry detail drawer showing the posting history timeline with a failed attempt and the error message]`

---

### `06-journal-log/exporting-journal-data.md`

**Audience:** Finance Officers.

**Content outline:**

- How to export the Journal Log: Journal Log → Export CSV. Apply date range and filter selections before exporting to narrow the data.
- What the export contains: all columns visible in the journal table plus the full entry lines (debit account, credit account, amount per line).
- Common uses: month-end reporting, sharing with external auditors, reconciling Ledgerise output against accounting system records.
- The journal-csv outbound adapter: if your accounting system has no API integration with Ledgerise, the journal-csv adapter can export journal entries in a format suitable for manual import. Configure it in Settings → Adapters.
- Filtering before export: filter by date range, product line, biller, currency, or status before exporting to get exactly the data you need.

`[SCREENSHOT: Journal Log Export CSV dialog showing filter options before downloading]`

---

## 07-settings/

Covers system configuration. Audience is primarily Admins, with Finance Officers having read-only access to most tabs.

---

### `07-settings/adapters.md`

**Audience:** Admins, Finance Officers (read-only).

**Content outline:**

- What the Adapters tab shows: a list of all registered inbound and outbound adapters. For each adapter: name, mode, version, healthcheck status, last run time, latency, and enable/disable toggle.
- Healthcheck status: Ledgerise checks that each adapter can reach its source system before the first run. If a healthcheck fails, the adapter is marked inactive and a detailed error message is shown.
- Enabling/disabling an adapter: use the toggle. Disabling an adapter stops all new data from that source. Existing records are not affected.
- Configuring adapter credentials: click "Configure" on any adapter. Enter the credentials your adapter needs (API keys, secrets, endpoint URLs). Credentials are stored encrypted using your LEDGERISE_CREDENTIALS_KEY.
- Uploading a file via CSV import: in the csv-import adapter, click "Upload File" to open the import flow.
- MVP default adapters:
  - **Inbound:** webhook-adapter, csv-import, poll-adapter.
  - **Outbound:** zoho-books, journal-csv.

`[SCREENSHOT: Settings > Adapters showing the adapter list with healthcheck status (green OK / red error), last run time, and the Configure button]`

---

### `07-settings/users-and-roles.md`

**Audience:** Admins.

**Content outline:**

- Where to manage users: Settings → Users.
- User list columns: name, email, role, last login, status (active/deactivated).
- The three roles and what each can do:

| Role | Transactions | Reconciliation | Mapping Rules | Journal Log | Settings |
|---|---|---|---|---|---|
| Admin | Full | Full | Full | Full | Full |
| Finance | Full | Full | Full | Full | Read-only |
| Auditor | Read-only | Read-only | No access | Read-only | Audit Log only |

- How to invite a user: Settings → Users → Invite User. Enter email and assign a role. The user receives an email invitation with a sign-in link.
- How to change a role: click the Edit icon on the user row → select a new role in the modal → confirm.
- How to deactivate a user: click Deactivate. The user's session is invalidated immediately. Their historical actions remain in the audit log.
- Best practice: every team member should have their own named account. Sharing login credentials is strongly discouraged and makes the audit trail unreliable.

`[SCREENSHOT: Settings > Users showing the user list with role badges and the Invite User button]`

`[SCREENSHOT: Role picker modal showing the three role options with descriptions]`

---

### `07-settings/system-settings.md`

**Audience:** Admins.

**Content outline:**

**System tab:**
- **Engine schedule** (cron expression): how often the journal engine runs. Default is every hour (`0 * * * *`). Change this in Settings → System. Lower intervals mean faster posting; higher intervals reduce API load on your accounting system.
- **Batch size**: the maximum number of transactions the engine processes per run. Default 500. Increase if you have high volume and your infrastructure can handle it.
- **Suspense account code**: the COA account code where unmapped transactions are posted. Configure this before going live. Make sure it matches an account in your accounting system.
- **Retry policy**: maximum retry attempts and backoff strategy for failed journal entries.

**License and environment:**
- Production activation is done in Settings → System. Enter your commercial license key and public key. The deployment switches from Sandbox to Production mode.
- The Sandbox badge in the top chrome shows whether you are in sandbox or production mode.

**Report sources:**
- Manage your saved reconciliation report sources (counterparty name + statement type combinations) from Settings.
- Report sources are created during the Import Statement flow and can be edited here.

`[SCREENSHOT: Settings > System tab showing the engine schedule, batch size, and suspense account fields]`

`[SCREENSHOT: Settings > System showing the license key input area and the sandbox/production mode indicator]`

---

### `07-settings/audit-log.md`

**Audience:** Admins, Auditors.

**Content outline:**

- What the Audit Log is: a system-wide, immutable event log covering all user and system actions in Ledgerise.
- Where to find it: Settings → Audit Log.
- What it records: mapping rule created / updated / activated / deactivated; exception resolved; break resolved; flag assigned / resolved; manual posting override; reconciliation import; user invited / role changed / deactivated; system settings changed.
- Columns: timestamp, event type, actor (user or system), target (rule ID, transaction ID, break ID, etc.), summary.
- Filtering: by event type, actor, and date range.
- The log is append-only: entries cannot be deleted or edited through the UI or any API. This is by design.
- How Auditors use it: Auditors have access to the Audit Log only within Settings. They can filter to a date range and review all system actions during a period of interest.

`[SCREENSHOT: Settings > Audit Log showing event rows with type, actor, target, and timestamp columns, and the filter bar at top]`

---

## 08-adapters/

For Admins configuring existing adapters and for developers building new ones.

---

### `08-adapters/overview.md`

**Audience:** Admins and developers.

**Content outline:**

- What adapters are and why they exist: Ledgerise has no built-in knowledge of Paystack, Flutterwave, Zoho Books, or any other system. Adapters are the only layer that does. This keeps the core engine stable while allowing the ecosystem to grow.
- Two directions: inbound adapters bring transaction data in; outbound adapters push journal entries out.
- Four inbound modes: webhook, file import, poll, and manual entry.
- MVP built-in adapters and what each is for.
- External/community adapters: developers can build adapters for any source or target system without changing the core engine. Link to building-an-adapter.md.
- All adapters expose the same interface (meta, validate, normalize, healthcheck) and speak the same canonical transaction schema.

---

### `08-adapters/adapter-spec.md`

**Audience:** Developers building a new adapter.

**Content outline:**

This is the developer-facing technical specification. Covers the full adapter interface that all adapters must implement.

- The four required methods: `adapter.meta()`, `adapter.validate(input)`, `adapter.normalize(input)`, `adapter.healthcheck()`.
- Input and output envelopes (success and failure).
- Normalize method rules (the 10 rules all adapters must follow).
- Standard error codes.
- Configuration: how to declare required config keys and how they are injected at runtime.
- Naming convention: `{source-system}-{mode}`.
- Testing requirements: required test cases and fixture files.
- Registration: how the engine discovers and loads adapters.

This page surfaces the same content as the existing `ADAPTER-SPEC.md` but written for the documentation audience (with context, examples, and explanations alongside the technical rules).

`[SCREENSHOT: Example adapter.meta() response in a code block with field annotations]`

---

### `08-adapters/building-an-adapter.md`

**Audience:** Developers.

**Content outline:**

A practical walkthrough for building a new inbound adapter from scratch.

- When to build a custom adapter vs. using a generic one: use the generic webhook/poll/csv adapters for systems that output standard JSON or CSV. Build a custom adapter when the source system has proprietary formats, custom authentication, or specific deduplication logic.
- Step 1: decide on the adapter mode (webhook, poll, or file import).
- Step 2: implement `adapter.meta()` — fill in name, version, source_system, modes, currency_codes, docs_url.
- Step 3: implement `adapter.validate(input)` — check for required fields and valid formats. Return a clean error list.
- Step 4: implement `adapter.normalize(input)` — call validate first; map source fields to the canonical schema; generate UUID for `id`; set `processed_at`; convert amounts to smallest currency unit; mask PII.
- Step 5: implement `adapter.healthcheck()` — for poll adapters, make a lightweight API call; for webhook adapters, return ok.
- Step 6: write tests (at minimum: valid completed transaction, failed transaction, missing fields, test environment).
- Step 7: add fixture files in `/fixtures`.
- Step 8: write a README for your adapter.
- Step 9: register the adapter (link to adapter-spec.md § Registration).

`[SCREENSHOT: Example normalize() implementation showing field mapping from a Paystack webhook payload to the canonical schema]`

---

### `08-adapters/generic-webhook.md`

**Audience:** Admins integrating a transaction source via webhook.

**Content outline:**

- Overview of the generic webhook adapter and when to use it.
- How it works: you configure a field mapping in the UI that tells the adapter how to find each canonical field in your webhook payload.
- Configuration fields: endpoint path, signing secret (for payload verification), field mapping configuration.
- Supported payload formats: JSON (single record or array of records).
- Testing: how to send a test webhook and verify the result.
- Limitations: when the generic webhook adapter is not enough (complex transformation logic, non-JSON formats) and when to build a custom adapter.

---

### `08-adapters/generic-csv.md`

**Audience:** Admins and Finance Officers.

**Content outline:**

- Overview of the generic CSV adapter.
- Supported file formats: CSV and XLSX.
- Configuration: column mapping, date format, amount format (full currency unit vs. smallest unit), delimiter.
- The import flow from the UI (link to transactions/csv-import.md for the step-by-step).
- How the column mapping is saved and reused across imports from the same source.

---

### `08-adapters/generic-poll.md`

**Audience:** Admins.

**Content outline:**

- Overview of the generic poll adapter.
- Configuration: API endpoint URL, authentication method, response field path, poll schedule, column-to-field mapping.
- How the poll cursor works and why it matters.
- Monitoring poll runs.
- Troubleshooting: what to do if a poll run fails or misses records.

---

### `08-adapters/zoho-books.md`

**Audience:** Admins.

**Content outline:**

- Overview: the Zoho Books outbound adapter posts Ledgerise journal entries to Zoho Books as manual journals.
- Prerequisites: a Zoho Books account, an OAuth 2.0 application configured in the Zoho Developer Console, and your organization ID.
- Configuration: Settings → Adapters → zoho-books → Configure. Enter Client ID, Client Secret, and Organization ID.
- OAuth flow: how the authorization flow works and where to find your credentials.
- How Ledgerise maps entries to Zoho: each Ledgerise journal entry becomes one Zoho Books manual journal record.
- Rate limits: Zoho allows 100 API calls per minute. Ledgerise batches entries and handles rate limiting automatically.
- COA mapping: Ledgerise account codes must match Zoho Books account names or codes. Sync your COA in Settings → COA Reference to keep them aligned.
- Troubleshooting common errors: AUTH_FAILED, rate limit errors, account code not found.

`[SCREENSHOT: zoho-books adapter Configure panel showing Client ID, Client Secret, and Organization ID fields]`

---

### `08-adapters/journal-csv-export.md`

**Audience:** Admins and Finance Officers whose accounting system has no API integration.

**Content outline:**

- Overview: the journal-csv outbound adapter exports generated journal entries as a CSV file for manual import into any accounting system.
- When to use it: when your accounting system is not yet supported by a Ledgerise API adapter (e.g., QuickBooks, Wave, Sage).
- Configuration: Settings → Adapters → journal-csv → Configure. Options: date format, amount display unit, filename pattern, whether to include source transaction IDs and mapping rule IDs.
- How to export: Journal Log → Export CSV (this uses the journal-csv adapter).
- What the CSV contains: one row per journal line (debit or credit), with journal ID, transaction ID, date, account code, amount, and currency.
- Importing into your accounting system: consult your accounting system's documentation for the journal import format. You may need to reformat the CSV.

---

## 09-licensing/

For Admins and commercial clients understanding the licensing model.

---

### `09-licensing/overview.md`

**Audience:** Admins and procurement stakeholders.

**Content outline:**

- Ledgerise is dual-licensed: the open-source version is available under Apache 2.0 (free to self-host). The commercial on-premise deployment requires a separate commercial license.
- What the commercial license gives you: a versioned Docker image from the private registry, implementation support, and a version support window.
- Ledgerise does not host your data. Your data stays in your infrastructure.
- The commercial license is based on monthly imported source rows (the number of transactions Ledgerise ingests per month).
- Usage is counted inside your own deployment — Ledgerise receives summary-only counts (no transaction rows, amounts, or customer identities).
- When a license expires or exceeds its limit, the deployment moves to read-only mode: all data remains accessible but ingestion and posting are suspended until a new license is activated.

---

### `09-licensing/commercial-tiers.md`

**Audience:** Admins and procurement stakeholders.

**Content outline:**

- Commercial tiers based on monthly imported source rows:

| Tier | Monthly imported source rows |
|---|---|
| Starter | Up to 30,000 |
| Pro | Up to 100,000 |
| Scale | Up to 300,000 |
| Enterprise | Custom volume above 300,000 |

- How to check your current usage: `/healthcheck` returns the current-month transaction count and your limit after production activation. Also visible in Settings → System.
- What happens when you approach your limit: Ledgerise refreshes license state daily. When usage exceeds the limit, the deployment moves to read-only on the next refresh.
- Upgrading your tier: contact Ledgerise. A new license key is issued and delivered via the standard delivery flow.
- Implementation fee: charged separately for on-premise deployments. Covers scoping, adapter configuration, mapping rule setup, and go-live support.

---

### `09-licensing/activating-your-license.md`

**Audience:** Admins.

**Content outline:**

- When to activate: after completing sandbox setup, testing, and inviting your team. Do not activate before you are ready for real production data.
- Before activation: ensure sandbox data has been reset (Settings → System → Reset sandbox data).
- How the key is delivered: Ledgerise sends a one-time retrieval link by email. Your client ID is sent separately (by phone, WhatsApp, or direct message). Visit the link, enter your client ID, and copy the key. The page expires after you copy the key.
- Activation steps:
  1. Settings → System.
  2. Enter your commercial license key.
  3. Enter the license public key (provided alongside the license key).
  4. Save.
  5. Confirm the Sandbox badge disappears and the status reads Production License.
  6. Confirm `/healthcheck` returns `environment_mode: "production"` and a valid license state.
- Treat the license key like a database password: enter it only in Settings → System. Do not put it in code, config files, or chat messages.

`[SCREENSHOT: Settings > System showing the license key and public key input fields with the activation button]`

`[SCREENSHOT: Settings > System after successful activation showing Production License status and environment mode]`

---

### `09-licensing/license-renewal-and-reissuance.md`

**Audience:** Admins.

**Content outline:**

- When a license expires: the deployment moves to read-only mode at the next daily refresh. Data is still accessible. Ingestion and posting are suspended.
- How to renew: contact Ledgerise. A new key is issued via the same delivery flow.
- If you lose your key before copying it: contact Ledgerise. The existing key is revoked and a new one is issued.
- If you suspect a key has been compromised: contact Ledgerise immediately. The key can be revoked and replaced.
- Reissuance flow: revocation takes effect at the next license refresh. Your deployment moves to read-only mode until the new key and public key are entered in Settings → System.

---

## 10-data-management/

For Admins responsible for backups, restores, and data retention.

---

### `10-data-management/backups.md`

**Audience:** Admins.

**Content outline:**

- Ledgerise uses BYODB (Bring Your Own Database). You own your database, and therefore you own your backups.
- Recommended backup strategy: automated daily snapshots of your PostgreSQL database to an object storage bucket (e.g., AWS S3, GCS, or a local NAS). Retain daily backups for 30 days; monthly backups for 12 months.
- How to back up PostgreSQL: `pg_dump` command with example. How to store the output securely.
- Encrypted backups: recommended for any deployment that handles real financial data.
- How often to back up: daily at minimum. For high-volume operators, consider hourly WAL archiving with point-in-time recovery.

`[SCREENSHOT: Example terminal command showing pg_dump and the resulting backup file]`

---

### `10-data-management/restore.md`

**Audience:** Admins.

**Content outline:**

- When to restore: disaster recovery, test environment setup, or backup verification.
- How to restore from a pg_dump backup: `pg_restore` or `psql` command with example.
- After a restore:
  1. Start the Ledgerise API against the restored DATABASE_URL.
  2. Confirm `/healthcheck` returns 200 with `repository: postgres` and `db: ok`.
  3. Verify sample record counts in the Transactions, Journal Log, and Reconciliation tabs.
- Restore drill cadence: run at least one restore drill per quarter. A backup that has never been tested is not a backup.
- Point-in-time recovery: if your PostgreSQL provider supports WAL archiving, you can restore to a specific timestamp. Consult your database provider's documentation.

---

### `10-data-management/retention-policy.md`

**Audience:** Admins, Finance Officers.

**Content outline:**

- Regulatory requirement: journal entries and transaction records must be retained for a minimum of 7 years under Nigerian CAMA and equivalent regulations in other target markets.
- Ledgerise does not automatically purge data. Retention policy is your responsibility.
- Reconciliation evidence: reconciliation run data, match records, break records, and evidence packages should be retained in line with your regulatory obligations.
- What not to delete: the `mapping_rule_audit` and `audit_log` tables are append-only and should never be purged during the retention window.
- Archiving strategy: after the active retention window, consider moving older records to cold storage rather than deleting them, so they remain accessible if needed.

---

## 11-security/

For Admins responsible for securing a Ledgerise deployment.

---

### `11-security/access-control.md`

**Audience:** Admins.

**Content outline:**

- Authentication: all Ledgerise dashboard access requires a username and password. There is no anonymous access.
- Session management: sessions expire after 8 hours of inactivity. Users are prompted to log in again.
- Role-based access control: three roles — Admin, Finance, and Auditor. Covered in full in settings/users-and-roles.md.
- User account best practices: every team member gets a named account. Shared credentials are not permitted. Deactivate accounts for team members who leave.
- API keys for webhook adapters: each webhook adapter has its own signing secret. Rotate these secrets independently in Settings → Adapters. Rotation does not require downtime — configure the new secret in the adapter config before updating your source system.

---

### `11-security/data-protection.md`

**Audience:** Admins.

**Content outline:**

- Credentials are encrypted at rest using AES-256-GCM. The key is your LEDGERISE_CREDENTIALS_KEY environment variable. This key must never be stored in source control.
- PII masking: Ledgerise stores no raw PII. Phone numbers are stored with only the last 4 digits visible. Card numbers: last 4 digits only. BVN and NIN: never stored.
- Data in transit: all API and dashboard traffic must be served over HTTPS with TLS 1.2 or higher. Configure TLS at your reverse proxy (nginx, Caddy, or a load balancer).
- Database connections: use SSL/TLS between the Ledgerise API and your PostgreSQL database. Most managed database providers enforce this by default.
- AI keys: if you use AI features, your AI provider key (e.g., Anthropic API key) is stored encrypted alongside your adapter credentials. Ledgerise does not use your key for anything outside your deployment.
- What Ledgerise sends externally: only summary-only usage counts during license check-ins. No transaction rows, amounts, customer identities, or financial records are sent outside your deployment.

---

### `11-security/webhook-security.md`

**Audience:** Admins and developers integrating source systems via webhook.

**Content outline:**

- Why webhook security matters: webhook endpoints are public URLs that receive financial data. Without signature verification, anyone can send forged payloads.
- How Ledgerise handles it: each inbound webhook adapter verifies the signature on every incoming payload. Payloads that fail verification are rejected with a 401 and are never normalized or stored.
- Provider-specific signature verification: payment providers such as Paystack and Flutterwave include a signature header (e.g., `x-paystack-signature`). Configure the signing secret in Settings → Adapters → your adapter → Configure.
- Replay attack protection: Ledgerise rejects webhook payloads with a timestamp older than 5 minutes. This prevents an attacker from replaying captured payloads.
- Rotating a webhook secret: update the secret in Settings → Adapters before updating the secret in your source system. Both systems should accept the new secret without a gap in webhook delivery.

---

## 12-reference/

Technical reference material. Not task-oriented — readers come here to look up a specific value or definition.

---

### `12-reference/transaction-schema.md`

**Audience:** Finance Officers, Developers, Auditors.

**Content outline:**

Complete field-by-field reference for the canonical transaction schema.

- Schema version and what version changes mean.
- Required fields table: field name, type, description.
- Optional fields table: field name, type, description.
- Field rules and constraints (e.g., amount is always a positive integer in the smallest currency unit; currency is always ISO 4217; source_id is used for deduplication).
- Where the schema is enforced: inbound adapters produce it, the journal engine reads it, outbound adapters consume journal entries generated from it.

---

### `12-reference/transaction-types.md`

**Audience:** Finance Officers and Developers.

**Content outline:**

Complete reference for the standard transaction type taxonomy. Covers all 80 standard types across 10 categories.

For each category: a table of type values, their subcategory structure, and a plain-English description of each.

Categories:
- `payment` (airtime, data, electricity, water, cable-tv, internet, insurance-premium, government-levy, education, transport, betting, merchant, invoice, subscription)
- `transfer` (wallet-to-wallet, wallet-to-bank, bank-to-wallet, agent-to-wallet, wallet-to-agent, internal)
- `collection` (pos, web, mobile, ussd, qr, nfc, api, agent, bank-transfer, direct-debit)
- `fee` (platform, processing, withdrawal, maintenance, card, fx, late-payment, reversal, chargeback)
- `loan` (disbursement, repayment.principal, repayment.interest, repayment.penalty, repayment.fee, write-off, provision, restructure)
- `savings` (deposit, withdrawal, interest-credit, liquidation)
- `investment` (purchase, maturity, yield-payout, liquidation)
- `remittance / fx` (remittance.send, remittance.receive, remittance.fee, fx.conversion, fx.fee, fx.gain, fx.loss)
- `card` (load, spend, reversal, chargeback, chargeback-reversal, fee, expiry-credit)
- `agency` (cash-in, cash-out, commission, vault-deposit, vault-withdrawal, float-allocation, float-recovery)
- `system` (reversal, refund, adjustment, settlement-batch, suspense-debit, suspense-credit, opening-balance, closing-balance)

Custom types: how to define and use a transaction type not in the standard taxonomy.

---

### `12-reference/error-codes.md`

**Audience:** Developers and Admins troubleshooting adapter errors.

**Content outline:**

Standard error codes returned by adapters in failure envelopes.

| Code | Description | Common cause | Suggested action |
|---|---|---|---|
| `VALIDATION_FAILED` | Input failed schema validation | Required field missing or wrong type | Check the `errors` array for field-level detail |
| `SOURCE_UNREACHABLE` | Could not connect to the source system | Network issue or provider downtime | Check healthcheck; verify network rules |
| `AUTH_FAILED` | Credentials rejected by the source system | Expired or incorrect API key | Re-enter credentials in Settings → Adapters |
| `RATE_LIMITED` | Source system returned a rate limit response | Too many requests | Ledgerise retries automatically; reduce poll frequency if recurring |
| `MALFORMED_PAYLOAD` | Input could not be parsed at all | Invalid JSON or corrupt file | Review the source payload; check for encoding issues |
| `UNSUPPORTED_EVENT` | The event type from the source system is not mapped | The provider sent a new event type the adapter does not handle | Update the adapter or add a mapping |
| `METHOD_NOT_SUPPORTED` | The called method is not supported by this adapter | The adapter does not support this mode | Use an adapter that supports the required mode |
| `ADAPTER_*` | Custom adapter-specific error | Varies — see the specific adapter's README | |

---

### `12-reference/glossary.md`

**Audience:** Everyone.

**Content outline:**

A single-page glossary of all Ledgerise terms, in alphabetical order. Longer than the key-concepts page in getting-started — this one is a lookup reference, not an introduction.

Terms (minimum):
- Adapter (inbound, outbound)
- Agent banking
- Biller / biller category
- Canonical schema
- Chart of Accounts (COA)
- Cursor (poll adapter)
- Double-entry bookkeeping
- Engine run
- Evidence package
- Float / float account
- Journal entry
- License key
- Mapping rule
- Match record
- Operator
- Outbound adapter
- Posting status
- Principal (customer, merchant, agent)
- Product line
- Reconciliation break
- Reconciliation case
- Reconciliation run
- Report source
- Sandbox mode
- Source environment (live vs. test)
- Source ID (deduplication key)
- Suspense account
- Transaction status
- Unmapped transaction
- Version (rule versioning)

---

## page count summary

| Section | Pages |
|---|---|
| getting-started/ | 4 |
| deployment/ | 6 |
| transactions/ | 5 |
| reconciliation/ | 6 |
| mapping-rules/ | 5 |
| journal-log/ | 4 |
| settings/ | 4 |
| adapters/ | 8 |
| licensing/ | 4 |
| data-management/ | 3 |
| security/ | 3 |
| reference/ | 4 |
| **total** | **56 pages** |

---

## screenshot summary

A separate design/asset pass is needed to capture the following screenshots from a live or demo Ledgerise deployment. All screenshots should be taken at 1440×900px with sample data that looks realistic (not lorem ipsum). Annotate screenshots with numbered callouts where the body text references specific UI elements.

**getting-started:** 4 screenshots
**deployment:** 6 screenshots
**transactions:** 7 screenshots
**reconciliation:** 11 screenshots
**mapping-rules:** 5 screenshots
**journal-log:** 5 screenshots
**settings:** 6 screenshots
**adapters:** 3 screenshots
**licensing:** 4 screenshots
**data-management:** 1 screenshot
**security:** 0 (command-line content only)
**reference:** 0 (tabular/text content only)

**Total screenshots needed: ~52**
