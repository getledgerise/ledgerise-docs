# Ledgerise documentation

Ledgerise is customer-managed payment operations infrastructure that automatically translates completed transactions into double-entry journal entries in your accounting system. You run it on your own servers — Ledgerise does not host your data.

This documentation covers everything needed to deploy, configure, and operate Ledgerise: initial deployment, adapter setup, mapping rules, reconciliation, journal review, and ongoing administration.

---

## quick links

### I'm a Finance Officer

| Task | Go to |
|---|---|
| Create or update a mapping rule | [Creating a rule](05-mapping-rules/02-creating-a-rule.md) |
| Import the chart of accounts | [Chart of accounts](05-mapping-rules/04-chart-of-accounts.md) |
| Import a bank or provider statement | [Importing a statement](04-reconciliation/02-importing-a-statement.md) |
| Review and resolve reconciliation breaks | [Resolving breaks](04-reconciliation/04-resolving-breaks.md) |
| Retry failed journal entries | [Retrying failed entries](06-journal-log/03-retrying-failed-entries.md) |
| Export journal data to a file | [Exporting journal data](06-journal-log/04-exporting-journal-data.md) |

### I'm an Admin

| Task | Go to |
|---|---|
| Deploy Ledgerise for the first time | [Docker deployment](02-deployment/02-docker-deployment.md) |
| Complete the go-live checklist | [Sandbox to production](02-deployment/05-sandbox-to-production.md) |
| Activate a commercial license | [Activating your license](09-licensing/03-activating-your-license.md) |
| Configure an adapter | [Adapters](07-settings/01-adapters.md) |
| Invite users and assign roles | [Users and roles](07-settings/02-users-and-roles.md) |
| Review the system audit log | [Audit log](07-settings/04-audit-log.md) |
| Upgrade to a new version | [Upgrading](02-deployment/06-upgrading.md) |

### I'm a Developer

| Task | Go to |
|---|---|
| Understand the adapter contract | [Adapter spec](08-adapters/02-adapter-spec.md) |
| Build a custom inbound adapter | [Building an adapter](08-adapters/03-building-an-adapter.md) |
| Look up the canonical transaction schema | [Transaction schema](12-reference/01-transaction-schema.md) |
| Look up transaction types | [Transaction types](12-reference/02-transaction-types.md) |
| Look up adapter error codes | [Error codes](12-reference/03-error-codes.md) |

### I'm an Auditor

| Task | Go to |
|---|---|
| View system and user activity | [Audit log](07-settings/04-audit-log.md) |
| Review journal entries | [Journal entries](06-journal-log/02-journal-entries.md) |
| Export journal data | [Exporting journal data](06-journal-log/04-exporting-journal-data.md) |
| Understand reconciliation evidence | [Evidence packages](04-reconciliation/07-evidence-packages.md) |

---

## table of contents

### 01 — getting started
- [Introduction](01-getting-started/01-introduction.md)
- [How Ledgerise works](01-getting-started/02-how-ledgerise-works.md)
- [Key concepts](01-getting-started/03-key-concepts.md)
- [Quickstart](01-getting-started/04-quickstart.md)

### 02 — deployment
- [Overview](02-deployment/01-overview.md)
- [Docker deployment](02-deployment/02-docker-deployment.md)
- [Environment variables](02-deployment/03-environment-variables.md)
- [First login](02-deployment/04-first-login.md)
- [Sandbox to production](02-deployment/05-sandbox-to-production.md)
- [Upgrading](02-deployment/06-upgrading.md)

### 03 — transactions
- [Overview](03-transactions/01-overview.md)
- [Ingestion methods](03-transactions/02-ingestion-methods.md)
- [Webhook adapter](03-transactions/03-webhook-adapter.md)
- [CSV import](03-transactions/04-csv-import.md)
- [Poll adapter](03-transactions/05-poll-adapter.md)
- [Transaction statuses](03-transactions/06-transaction-statuses.md)
- [Unmapped transactions](03-transactions/07-unmapped-transactions.md)

### 04 — reconciliation
- [Overview](04-reconciliation/01-overview.md)
- [Importing a statement](04-reconciliation/02-importing-a-statement.md)
- [Understanding matches](04-reconciliation/03-understanding-matches.md)
- [Resolving breaks](04-reconciliation/04-resolving-breaks.md)
- [Reconciliation rules](04-reconciliation/05-reconciliation-rules.md)
- [Generating reports](04-reconciliation/06-generating-reports.md)
- [Evidence packages](04-reconciliation/07-evidence-packages.md)

### 05 — mapping rules
- [Overview](05-mapping-rules/01-overview.md)
- [Creating a rule](05-mapping-rules/02-creating-a-rule.md)
- [Rule resolution order](05-mapping-rules/03-rule-resolution-order.md)
- [Chart of accounts](05-mapping-rules/04-chart-of-accounts.md)
- [Rule audit trail](05-mapping-rules/05-rule-audit-trail.md)

### 06 — journal log
- [Overview](06-journal-log/01-overview.md)
- [Journal entries](06-journal-log/02-journal-entries.md)
- [Retrying failed entries](06-journal-log/03-retrying-failed-entries.md)
- [Exporting journal data](06-journal-log/04-exporting-journal-data.md)

### 07 — settings
- [Adapters](07-settings/01-adapters.md)
- [Users and roles](07-settings/02-users-and-roles.md)
- [System settings](07-settings/03-system-settings.md)
- [Audit log](07-settings/04-audit-log.md)

### 08 — adapters
- [Overview](08-adapters/01-overview.md)
- [Adapter spec](08-adapters/02-adapter-spec.md)
- [Building an adapter](08-adapters/03-building-an-adapter.md)
- [Generic webhook](08-adapters/04-generic-webhook.md)
- [Generic CSV](08-adapters/05-generic-csv.md)
- [Generic poll](08-adapters/06-generic-poll.md)
- [Zoho Books](08-adapters/07-zoho-books.md)
- [Journal CSV export](08-adapters/08-journal-csv-export.md)

### 09 — licensing
- [Overview](09-licensing/01-overview.md)
- [Commercial tiers](09-licensing/02-commercial-tiers.md)
- [Activating your license](09-licensing/03-activating-your-license.md)
- [License renewal and reissuance](09-licensing/04-license-renewal-and-reissuance.md)

### 10 — data management
- [Backups](10-data-management/01-backups.md)
- [Restore](10-data-management/02-restore.md)
- [Retention policy](10-data-management/03-retention-policy.md)

### 11 — security
- [Access control](11-security/01-access-control.md)
- [Data protection](11-security/02-data-protection.md)
- [Webhook security](11-security/03-webhook-security.md)

### 12 — reference
- [Transaction schema](12-reference/01-transaction-schema.md)
- [Transaction types](12-reference/02-transaction-types.md)
- [Error codes](12-reference/03-error-codes.md)
- [Glossary](12-reference/04-glossary.md)
