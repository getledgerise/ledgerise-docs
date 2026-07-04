# licensing overview

Ledgerise is available as a **free online demo** for evaluation and as a **commercial on-premise license** for production deployments. Understanding which applies to your situation is the starting point for procurement and go-live planning.

---

## free online demo

The Ledgerise demo lets you explore the platform without setting up any infrastructure. Import sample transactions, configure mapping rules, and see how the reconciliation and journal engine work — all in a hosted demo environment maintained by Ledgerise.

The demo is intended for evaluation. It is not a production deployment: data does not persist between sessions, live adapter connections to payment systems are not supported, and journal entries are not posted to any accounting system.

Contact Ledgerise to access the demo environment.

---

## commercial on-premise license

The commercial license is required to run Ledgerise in a production environment. It gives you:

- Access to the private versioned Docker image in GitHub Container Registry. Ledgerise provides the `ledgerise-dev` registry username and a personal access token so the client can pull the image.
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

- [Commercial tiers](02-commercial-tiers.md) — tier limits, usage monitoring, and what to do when you approach your limit
- [Activating your license](03-activating-your-license.md) — step-by-step activation instructions
- [License renewal and reissuance](04-license-renewal-and-reissuance.md) — what to do when a license expires, is lost, or needs to be replaced
