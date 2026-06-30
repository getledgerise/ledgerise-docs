# glossary

Alphabetical reference for Ledgerise terminology. For conceptual introductions, see [key concepts](../01-getting-started/03-key-concepts.md).

---

**Adapter**
A self-contained module that translates data between an external system and Ledgerise's internal format. Inbound adapters normalize raw transaction data from a source system into the canonical schema. Outbound adapters receive journal entries from the engine and post or export them to an accounting system. Adapters have no knowledge of COA accounts or journal logic — that boundary is enforced by design.

→ See [adapters overview](../08-adapters/01-overview.md)

---

**Agent banking**
A distribution model in which a third-party agent (a shop owner, kiosk operator, or individual) performs financial transactions on behalf of customers using the operator's platform. Common transactions include cash-in, cash-out, and bill payment. Ledgerise supports agent banking transactions through the `agency.*` transaction type category.

---

**Biller**
The specific service provider that a payment transaction is directed to. For example, `IKEDC`, `DStv`, or `MTN` within a bill payment product. Biller is the second priority field in mapping rule resolution: rules can target a specific biller within a product line for precise COA mapping.

→ See [rule resolution order](../05-mapping-rules/03-rule-resolution-order.md)

---

**Biller category**
A grouping of billers within a product line. For example, all electricity distribution companies might be grouped under `electricity`, while all cable TV providers fall under `cable-tv`. Biller category is the third priority field in mapping rule resolution — less specific than an exact biller match, more specific than a product line catch-all.

---

**Canonical schema**
The internal format that every Ledgerise transaction record must conform to, regardless of its source system. The canonical schema is the contract between the inbound adapter layer and the journal engine. It defines required and optional fields, data types, and constraints.

→ See [transaction schema](01-transaction-schema.md)

---

**Chart of Accounts (COA)**
The structured list of accounting codes (accounts) maintained in your accounting system (Zoho Books or QuickBooks). In Ledgerise, the COA is imported as a read-only reference — it cannot be edited through Ledgerise. Finance Officers use the COA when creating mapping rules to assign debit and credit accounts to transaction types.

→ See [chart of accounts](../05-mapping-rules/04-chart-of-accounts.md)

---

**Catch-all rule**
A mapping rule that targets only a product line, with no biller or biller category specified. It matches any transaction in that product line that no more-specific rule covers. Every product line should have a catch-all rule to prevent unmapped transactions from accumulating in the suspense account.

---

**Cursor (poll adapter)**
A position marker that the poll adapter saves after each successful run. On the next run, the adapter uses the cursor to fetch only records that have arrived since the last successful poll, avoiding duplicate ingestion. The cursor typically contains the timestamp or record ID of the last fetched transaction.

---

**Deduplication**
The process by which the journal engine prevents the same transaction from generating multiple journal entries. The engine uses the `source_id` field to detect and skip records it has already processed. Adapters that cannot populate `source_id` lose this protection.

---

**Direction**
A field on every canonical transaction record indicating whether the transaction is a `debit` or `credit` from the operator's perspective. Amounts are always positive — direction is captured by this field rather than by the sign of the amount.

---

**Double-entry bookkeeping**
The accounting principle that every financial transaction must be recorded as at least one debit and one equal credit. Ledgerise generates double-entry journal entries for every posted transaction — a debit to one COA account and a credit to another. The journal engine validates that debits equal credits before submitting to the accounting system.

---

**Engine run**
One cycle of the journal engine. During a run, the engine fetches unposted transactions, applies mapping rules, generates journal entries, and submits them to the outbound adapter. Engine runs happen on a configurable schedule (set in Settings → System) or can be triggered manually from the Journal Log.

→ See [journal entries](../06-journal-log/02-journal-entries.md)

---

**Evidence package**
A generated document that captures the full reconciliation record for a transaction or reconciliation run — including match records, break records, resolution notes, and timestamps. Evidence packages are produced for regulatory or audit purposes and are stored in your deployment.

---

**Float / float account**
Float is the liquidity balance an operator maintains with an aggregator to fund real-time transactions. A float account records the movement of this balance. The `float.*` fields on canonical transaction records and the `agency.float-allocation` and `agency.float-recovery` transaction types are used to track float movements in the journal.

---

**Journal entry**
A double-entry accounting record generated by the journal engine for a completed transaction. Each journal entry contains a debit line and one or more credit lines, with COA account codes, amounts, and a reference to the originating transaction. Journal entries are the output Ledgerise submits to your accounting system.

→ See [journal entries](../06-journal-log/02-journal-entries.md)

---

**License key**
A cryptographic key that activates a commercial Ledgerise deployment and determines its tier and usage limits. The license key is delivered via a one-time secure retrieval link. It is entered in Settings → System to switch the deployment from sandbox to production mode.

→ See [activating your license](../09-licensing/03-activating-your-license.md)

---

**Mapping rule**
A configuration entry that tells the journal engine which COA accounts to debit and credit for a given combination of product line, biller, biller category, and transaction type. Mapping rules are created and managed by Finance Officers through the Mapping Rules page.

→ See [mapping rules overview](../05-mapping-rules/01-overview.md)

---

**Match record**
A record created during a reconciliation run that pairs one internal transaction record with one external statement line as a confirmed match. Match records are the primary output of the reconciliation engine and form the basis of the reconciliation evidence trail.

---

**Operator**
The company or organization running a Ledgerise deployment. In Ledgerise's model, the operator is you: you own the infrastructure, the database, and the deployment. Ledgerise does not host operator data or have access to operator systems.

---

**Outbound adapter**
An adapter that receives journal entries from the engine and pushes or exports them to an accounting system. The built-in outbound adapters are the Zoho Books adapter (real-time API posting) and the journal-csv adapter (export to file for manual import).

→ See [adapters overview](../08-adapters/01-overview.md)

---

**Posting gate**
A configurable rule that controls whether journal entries are submitted to the accounting system based on their reconciliation status. The four values are: `disabled` (post regardless of reconciliation status), `provider_matched_or_better`, `bank_matched_or_better`, and `full_match_or_resolved`. Default is `disabled`.

→ See [system settings](../07-settings/03-system-settings.md)

---

**Posting status**
The state of a journal entry with respect to submission to the accounting system. Values: `unposted` (waiting for the next engine run), `posted` (successfully submitted), `failed` (submission attempt failed), `duplicate` (skipped because the `source_id` was already posted), `pending_reconciliation` (held by the posting gate).

---

**Principal**
The customer, merchant, or agent involved in a transaction. In the canonical schema, the principal is identified by `principal.id` (an opaque internal ID, never PII) and optionally described by `principal.type` and a masked `principal.reference`.

---

**Product line**
The operator's internal grouping of transaction types — for example, `bill-payment`, `agent-banking`, `card-services`, or `lending`. Product line is the primary key for mapping rule resolution. Every canonical transaction record must have a `product.line` value.

---

**Reconciliation break**
A discrepancy identified during a reconciliation run where an internal record has no matching external statement line (or vice versa), or where a match was found but amounts or dates do not agree within tolerance. Breaks require manual review and resolution.

---

**Reconciliation case**
The record created for each reconciliation break that requires follow-up. A case tracks the break details, the investigation notes, the resolution applied, and who resolved it. Cases appear in the Reconciliation page.

---

**Reconciliation run**
One execution of the reconciliation engine against an imported bank or provider statement. During a run, the engine matches internal transaction records against external statement lines using exact, tolerance, fuzzy, and manual tiers. The output is match records, break records, and a run summary.

→ See [reconciliation overview](../04-reconciliation/01-overview.md)

---

**Report source**
A configured data source from which Ledgerise can import bank or provider statements for reconciliation. Examples: a specific bank account, a payment provider's settlement report, or a float account with an aggregator. Report sources are configured in Settings → System.

---

**Sandbox mode**
The default mode of a fresh Ledgerise deployment. In sandbox mode, the engine runs and generates journal entries, but they are never posted to any accounting system. Sandbox mode is safe for configuration, testing, and team training. It ends when you activate a commercial license.

→ See [first login](../02-deployment/04-first-login.md)

---

**Source environment**
A flag on each canonical transaction record indicating whether it came from a live production system (`live`) or a test environment (`test`). Test transactions are stored but never posted. Adapters must not default to `live` without explicit confirmation from the source data that the transaction is real.

---

**Source ID**
The original transaction identifier from the source system, stored in the `source_id` field. Used by the engine as the deduplication key — if two records arrive with the same `source_id`, only the first is processed. Adapters must set `source_id` to the most stable unique identifier the source system provides.

---

**Suspense account**
A holding account in your COA used to capture transactions that the engine could not map to a specific COA account. Transactions route to the suspense account when no mapping rule matches their product line, biller, and transaction type. Finance Officers review and reclassify suspense entries.

→ See [system settings](../07-settings/03-system-settings.md)

---

**Transaction status**
The state of a canonical transaction record with respect to the source system. Values: `pending` (not yet finalized), `completed` (finalized and eligible for journal posting), `failed` (the source transaction failed — stored but not posted), `reversed` (a previously completed transaction has been reversed), `disputed` (under dispute). Only `completed` transactions are eligible for journal posting.

---

**Unmapped transaction**
A transaction the engine could not map to COA accounts because no matching rule exists for its product line, biller, and type combination. Unmapped transactions are posted to the suspense account and surfaced in the Journal Log's **Unmapped** count and filter.

---

**Version (mapping rule)**
Every change to a mapping rule creates a new version rather than overwriting the previous one. The rule audit trail records all versions, who made each change, and when. The journal engine logs which rule version was applied to each journal entry, creating a complete chain of evidence between any journal entry and the rule configuration that generated it.

→ See [rule audit trail](../05-mapping-rules/05-rule-audit-trail.md)
