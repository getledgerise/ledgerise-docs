# licensing overview

Ledgerise is dual-licensed. Understanding which license applies to your deployment, and what the commercial license gives you, is the starting point for procurement and go-live planning.

---

## the two licenses

**Apache 2.0 (open source)**

The Ledgerise source code is available under the Apache 2.0 license. You can self-host, modify, and distribute it at no cost under those terms. The Apache 2.0 build does not include the private Docker image, the versioned release channel, or implementation support.

**Commercial on-premise license**

The commercial license is required to run Ledgerise in a production environment under a supported configuration. It gives you:

- A versioned Docker image pulled from the Ledgerise private registry — no build toolchain, no source code required.
- A supported version window with changelog and migration notes for each release.
- Implementation support: scoping, adapter configuration, mapping rule setup, and go-live assistance.

The commercial license is not SaaS. Ledgerise does not host your deployment, your database, or your data. Everything runs inside your own infrastructure.

---

## your data stays with you

Ledgerise is customer-managed infrastructure. Transaction records, journal entries, reconciliation data, mapping rules, and credentials are stored in your PostgreSQL database on your servers. Ledgerise never receives, stores, or processes your transaction data.

The only information that leaves your deployment is a summary-level usage count — the number of transactions ingested in the current month. No transaction rows, amounts, account codes, or customer identities are ever transmitted.

---

## how usage is counted

The commercial license is priced by **monthly imported source rows** — the number of transaction records Ledgerise ingests each calendar month across all inbound adapters. Each unique canonical transaction record counted once when it is first ingested.

Usage resets at the start of each calendar month. Your current-month count and your tier limit are visible in:

- `GET /healthcheck` — returns `license.usage.current_month` and `license.usage.limit` after production activation.
- **Settings → System** — the License section shows the same figures alongside your tier name and license state.

---

## what happens when a license expires or is exceeded

When your deployment exceeds its monthly limit, or when your license term expires, Ledgerise checks license state on its daily refresh and moves the deployment to **read-only mode**:

- All existing data remains fully accessible — the dashboard, all pages, all exports.
- New transaction ingestion is suspended.
- Journal entry posting is suspended.
- The Sandbox badge is replaced with a **License Required** notice in the top navigation bar.

Normal operation resumes as soon as a valid license key is entered in Settings → System.

---

## where to go next

- [Commercial tiers](commercial-tiers.md) — tier limits, usage monitoring, and what to do when you approach your limit
- [Activating your license](activating-your-license.md) — step-by-step activation instructions
- [License renewal and reissuance](license-renewal-and-reissuance.md) — what to do when a license expires, is lost, or needs to be replaced
