# retention policy

Ledgerise does not automatically purge or archive data. Retention policy is your responsibility as the operator. This page explains the regulatory baseline, what must not be deleted, and a recommended archiving approach.

---

## regulatory minimum

Under the Nigerian Companies and Allied Matters Act (CAMA) and equivalent regulations in other markets Ledgerise targets, financial records — including journal entries and transaction records — must be retained for a minimum of **7 years**.

This is a minimum. Some regulatory frameworks and accounting standards require longer retention for specific record types. Confirm the requirements that apply to your jurisdiction and industry before setting a deletion policy.

---

## what Ledgerise stores that is subject to retention

| Record type | Location | Minimum retention |
|---|---|---|
| Canonical transaction records | `transactions` table | 7 years |
| Journal entries and posting history | `journal_entries`, `posting_attempts` | 7 years |
| Mapping rule configurations | `mapping_rules` | 7 years |
| Mapping rule audit trail | `mapping_rule_audit` | 7 years |
| Reconciliation runs, match records, break records | `recon_runs`, `recon_matches`, `recon_breaks` | 7 years |
| Evidence packages | Generated documents stored on your server | 7 years |
| System-wide audit log | `audit_log` | 7 years |
| User accounts and role history | `users` | Duration of employment + standard HR retention |

---

## what must not be deleted during the retention window

Two tables are append-only by design and must never be manually purged or truncated:

**`mapping_rule_audit`** — records every version of every mapping rule ever active in your deployment. Deleting from this table severs the chain of evidence connecting a journal entry to the rule version that generated it. Auditors and regulators rely on this chain.

**`audit_log`** — records every user and system action. Deleting entries undermines the non-repudiation of the event record. The Ledgerise UI and API do not provide a delete action for this table — any deletion would require direct database access and should be treated as a serious control failure.

---

## reconciliation evidence

Reconciliation records have the same retention requirements as journal entries, since they are part of the evidence chain for any regulatory enquiry about a specific transaction:

- Reconciliation runs — which statement was imported, when, by whom, and the resulting match rate.
- Match records — confirmed pairs between internal and external records.
- Break records — discrepancies raised, who investigated, what resolution was applied, and when.
- Evidence packages — generated PDF/audit documents exported for specific transactions or breaks.

Do not delete reconciliation data within the retention window even if the underlying provider relationship has ended.

---

## active vs. archived records

Ledgerise does not distinguish between active and archived records in its data model — all records are stored in the same tables and are equally queryable. For operators accumulating several years of transaction volume, this can lead to large table sizes that affect query performance.

A recommended approach:

1. After a record passes the regulatory retention window, export it to cold storage (a compressed database dump or structured export to object storage) before deleting it from the live database.
2. Keep the exported archive accessible — do not assume data past the minimum retention window will never be needed.
3. Document your archiving decisions in your internal data governance policy so that you can demonstrate to regulators that records were retained for the required period before archival.

Work with your legal and compliance team to define the archiving boundary before making any deletions.

---

## backup retention vs. data retention

These are distinct:

**Backup retention** — how long you keep database snapshot files. This is an operational concern. → See [backups](backups.md)

**Data retention** — how long the records within the database must be kept. This is a regulatory and compliance concern.

A 30-day backup window does not satisfy a 7-year data retention requirement. Backups give you recovery capability; they are not a substitute for a retention and archiving policy that keeps individual records accessible for their full required lifetime.
