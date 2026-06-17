# Reconciliation Rules Guide

Status: Draft

This guide explains how reconciliation rules work in Ledgerise today, how they should be presented in the product, and what product decisions remain open before the rules experience is considered commercially ready.

## Purpose

Reconciliation rules tell the matching engine how to compare internal Ledgerise transactions with external processor, aggregator, or bank evidence.

They answer questions like:

- Which field on the internal transaction should be matched to the external report?
- How much amount variance is acceptable before a break is raised?
- How should processor statuses map to canonical transaction statuses?
- How many days of settlement delay should be tolerated?
- Which duplicate or direction cases should be flagged?

Rules are separate from journal mapping rules. Reconciliation rules decide whether evidence matches. Journal mapping rules decide how matched or approved transactions post into accounts.

## Rule Lifecycle

Each reconciliation rule has:

| Field | Meaning |
|---|---|
| `report_source_id` | Saved report source the rule applies to. Product display format is `Source Name — Report Name`, such as `Paystack — Settlement Report` or `GTBank — Collection Account Statement`. |
| `source_name` | Provider, processor, aggregator, switch, or bank name, such as `Paystack`, `Flutterwave`, or `GTBank`. |
| `report_type` | Broad report family: `Provider Report` or `Bank Statement`. |
| `report_name` | Report identity inside the family, such as `Transaction Report`, `Settlement Report`, `Fee Report`, or `Settlement Account Statement`. |
| `rule_category` | Functional bucket, such as `identity`, `amount`, or `fee`. |
| `rule_type` | Specific behavior inside the category. |
| `priority` | Ordering hint. Lower priority values should be evaluated first where ordering matters. |
| `description` | Product-facing explanation shown in the UI. This should be written for operators, not engineers. |
| `config` | JSON object containing rule-specific settings. |
| `status` | `draft`, `active`, or `inactive`. Only active rules affect matching. |
| `version` | Incremented when rule changes are published. |

### Statuses

| Status | Product meaning | Engine behavior |
|---|---|---|
| `draft` | Saved but not live. Useful for review. | Ignored. |
| `active` | Published and live. | Used by matching runs. |
| `inactive` | Retired but retained for audit. | Ignored. |

### Versioning

Rules are audit-sensitive because they can change match outcomes. Product decisions should assume:

- Publishing a rule change should create an audit event.
- Previous versions should remain visible.
- Runs should be explainable by the rule versions that were active at run time.
- Drafts should not alter run results.

## Engine Flow

The current deterministic matching engine evaluates rules in this broad order:

1. Build active rule set for the run's saved report source.
2. Scope internal candidate transactions using the import's product line, biller, biller category, and transaction type filters.
3. Apply Reference Matching rules.
4. Apply Account Mapping rules.
5. Apply Amount Tolerance rules.
6. Apply Fee Schedule rules.
7. Apply Timing rules.
8. Apply Status Mapping rules.
9. Apply Direction Check rules.
10. Apply Duplicate Detection rules.
11. Apply Escalation rules only where the workflow explicitly supports audited escalation or auto-resolution behavior.
12. Create match records or break records.

Important: if identity matching fails, downstream amount, fee, date, and status checks cannot run for that pair because the engine has no confirmed candidate pair.

## Category Map

The UI should use product-facing labels. The backend stores technical enum values.

| UI label | Backend `rule_category` | Main purpose |
|---|---|---|
| Reference Matching | `identity` | Link internal transactions to external records. |
| Account Mapping | `account_mapping` | Map bank/settlement accounts and GL or processor context. |
| Amount Tolerance | `amount` | Decide allowed amount variance. |
| Fee Schedule | `fee` | Validate reported fees or net settlement. |
| Timing | `date` | Allow or reject settlement/value date lag. |
| Status Mapping | `status` | Compare canonical status to processor/bank status. |
| Direction Check | `direction` | Validate debit/credit or inbound/outbound direction independently from status. |
| Duplicate Detection | `duplicate` | Identify repeated evidence. |
| Escalation | `auto_resolution` | Suppress, escalate, or auto-resolve eligible breaks only under explicit workflow controls. |

This order is fixed in the MVP UI. When two or more rules exist in the same category for the same saved report source, sort those rules by `priority` ascending.

## Report Sources

Rules attach to saved report sources, not broad source names alone. A report source is the saved combination of:

- Source Name
- Report Type
- Report Name
- Reconciliation Scope defaults

Product display format:

```text
Source Name — Report Name
```

Examples:

- `Paystack — Transaction Report`
- `Paystack — Settlement Report`
- `Flutterwave — Chargeback Report`
- `GTBank — Collection Account Statement`

Provider Report names should start with these presets:

- Transaction Report
- Settlement Report
- Switch Report
- Processor Report
- Aggregator Report
- Fee Report
- Chargeback Report
- Reversal Report
- Other

Bank Statement names should start with these presets:

- Settlement Account Statement
- Collection Account Statement
- Payout Account Statement
- Credit Report
- Debit Report
- Other

If the report name is not listed, the user selects `Other`, enters a custom report name, and saves the report source with the same `Source Name — Report Name` display format.

## Categories And Current Config

### Reference Matching (`identity`)

Reference matching is the most important rule category. Without a usable identity rule, the engine cannot reliably pair internal and external records.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `identity_exact` | Compare one internal field to one external field after normalization. |
| `prefix_suffix_strip` | Strip configured prefixes/suffixes before comparison. |
| `custom_field` | Match on a non-default internal/external field pair. |

Required config:

```json
{
  "internal_field": "source_id",
  "external_field": "reference",
  "case_sensitive": false
}
```

Optional config for strip behavior:

```json
{
  "internal_strip_prefix": "TXN-",
  "internal_strip_suffix": "",
  "external_strip_prefix": "PST-",
  "external_strip_suffix": ""
}
```

Product guidance:

- Make every source configure at least one reference matching rule.
- Default to `source_id` to `reference`.
- Let operators preview how sample rows normalize before publishing.
- Avoid fuzzy matching as a default until the UI can explain confidence and false-positive risk.

### Amount Tolerance (`amount`)

Amount rules run after identity matching. They compare internal amount to external gross amount.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `amount_exact` | No variance allowed. |
| `amount_tolerance` | Allow absolute and/or percentage tolerance. |

Config:

```json
{
  "absolute_tolerance": 50,
  "percentage_tolerance": 0.1
}
```

Behavior:

- If no amount tolerance rule exists, tolerance is `0`.
- If both absolute and percentage tolerance are set, the engine uses the larger allowance.
- Differences inside tolerance are matched with `tolerance_approved`.
- Differences outside tolerance raise `amount_mismatch`.

Product guidance:

- Show amounts in major currency units in the UI, but store smallest units.
- Require Finance approval for broad percentage tolerances.
- Explain whether tolerance is absolute, percentage, or both.

### Fee Schedule (`fee`)

Fee rules validate fee or net-settlement evidence. They are especially important for processor reports where gross amount may match but net settlement differs because of fees.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `fee_exact` | Compare internal fee to external fee. |
| `net_after_fee` | Compute expected net and compare with external net. |

For `net_after_fee`, the rule drawer collects an inline fee schedule. The user first chooses a fee model, then only fills the fields that apply to that model:

```json
{
  "external_fee_field": "fee",
  "external_net_field": "net_amount",
  "fee_basis": "gross_amount",
  "fee_model": "percentage_plus_flat",
  "percentage_rate": 1.5,
  "flat_fee": 100,
  "minimum_fee": 0,
  "cap_amount": 2000,
  "waiver_threshold": 2500,
  "currency": "NGN",
  "absolute_tolerance": 0
}
```

Fee model options:

| Fee model | Meaning |
|---|---|
| `percentage` | Fee is calculated as a percentage of the selected basis. |
| `flat` | Fee is a fixed amount in the smallest currency unit. |
| `percentage_plus_flat` | Fee combines a percentage and a fixed amount. |

Behavior:

- `fee_exact` raises `fee_mismatch` if internal and external fee differ.
- `net_after_fee` computes `expected_net = internal_amount - expected_fee`.
- Expected fee comes from the internal canonical fee if present, otherwise from the inline fee schedule.
- Existing legacy rules with `fee_schedule_ref` can still fall back to active fee schedules.

Product guidance:

- Treat fee schedule changes as finance-owned rule changes.
- Show fee rule descriptions in plain English, such as "1.5% capped at NGN 2,000, waived below NGN 2,500."
- Legacy `fee_schedule_ref` values remain visible as legacy schedule references. New rules should use inline fee model fields instead.

### Timing (`date`)

Timing rules check whether external settlement/value dates are close enough to the internal transaction date.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `settlement_window` | Compare internal settlement date to external date. |
| `value_date_window` | Same behavior with bank-style value date language. |

Config:

```json
{
  "internal_field": "completed_at",
  "external_field": "settlement_date",
  "max_days": 1,
  "calendar_policy": "calendar_days"
}
```

Behavior:

- The drawer labels `internal_field` as the Ledgerise transaction date. New rules default it to `completed_at`.
- Settlement window rules prefer settlement-like or record-date report fields such as `settlement_date`, `settled_on`, or `record_date`.
- Value date window rules prefer value-date report fields such as `value_date`.
- Value date means the bank/accounting date funds become available.
- If the configured external date field is absent, the engine falls back to normalized imported record/value dates.
- If no external date is available, no timing break is raised by this check.
- If date difference exceeds `max_days`, the engine raises `timing_difference`.
- `calendar_policy` can be `calendar_days` or `business_days`; business days exclude Saturdays and Sundays.

Product guidance:

- Configure different windows for processors and banks.
- Nigerian settlement operations may need holiday calendars later; current business-day support excludes weekends only.
- Add holiday calendar support before selling complex bank reconciliation promises.

### Status Mapping (`status`)

Status mapping rules compare canonical transaction status to source-specific external statuses.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `status_map` | Map internal statuses to allowed external statuses. |

Config:

```json
{
  "external_status_field": "status",
  "status_map": {
    "completed": ["success", "successful", "settled", "00"],
    "failed": ["failed", "declined"],
    "reversed": ["reversed", "refunded"]
  }
}
```

Behavior:

- If external status exists and is not in the allowed list for the internal status, the engine raises `status_mismatch`.
- If no mapping exists for the internal status, the rule does not raise a mismatch.

Product guidance:

- Choose the external status column from the saved report mapping.
- Edit allowed values as chips/lists; avoid asking users to type comma-separated config.
- Provide source-specific presets for Paystack, Flutterwave, Interswitch, and common banks.
- Make status values case-insensitive.
- Let operators import statuses from sample files to reduce manual typing.

### Direction Check (`direction`)

Direction Check rules validate debit/credit, inbound/outbound, or report-specific direction semantics independently from status mapping.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `direction_check` | Ensure internal and external directions match expected values. |

Config:

```json
{
  "expected_internal_direction": "credit",
  "external_direction_field": "direction",
  "expected_external_direction": "credit"
}
```

Behavior:

- If internal direction differs from `expected_internal_direction`, the engine raises `direction_mismatch`.
- If the external direction field is present and differs from `expected_external_direction`, the engine raises `direction_mismatch`.

Product guidance:

- Keep Direction Check separate from Status Mapping in the UI.
- Choose the external direction column from the saved report fields. Use the common `direction` field when the saved mapping does not yet store a dedicated direction column.
- Use standard direction values where possible: credit, debit, inbound, outbound, CR, or DR.
- Use this category for debit/credit semantics, payout/collection direction, and bank statement credit/debit expectations.
- Account Mapping may store directional metadata, but Direction Check owns the matching validation.

### Duplicate Detection (`duplicate`)

Duplicate detection protects the matching pool from repeated external evidence.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `duplicate_identity` | Detect duplicate external records by normalized identity. |

Current behavior:

- Duplicate external records are grouped by the first available identity value.
- If more than one external record has the same identity, those records produce `duplicate_external` breaks.

Product guidance:

- Choose the duplicate identity field from saved report fields, usually the reference column.
- Be clear that current engine duplicate detection is identity-based.
- Future product work should support duplicate windows and duplicate amount/date constraints.

### Escalation / Auto Resolution (`auto_resolution`)

Auto-resolution is intended for low-risk exceptions that should not require manual review every time.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `manual_only` | Explicitly prevent automatic closure. |
| `suppress_duplicate` | Intended to suppress eligible duplicate breaks. |

Config:

```json
{
  "eligible_break_types": ["duplicate_external"],
  "max_amount_minor": 0,
  "confidence_threshold": 0.95,
  "action": "suppress"
}
```

Current engine note:

- The category is validated and stored, but broad automatic break closure remains a product and workflow decision.
- Break resolution APIs still require resolution type and note.

Product guidance:

- Start conservative. Make `manual_only` the default.
- Choose eligible break types from the controlled list. Duplicate suppression is limited to duplicate break types.
- Show `max_amount_minor` as a major-unit amount in the UI with the helper text "Auto-handle only if difference is at most this amount."
- Explain that suppressed duplicate external records do not create standalone journal entries.
- Keep suppressed duplicate breaks auditable with the `duplicate_suppressed` resolution type.
- Require audit copy for every automatic resolution.
- Put auto-resolution behind role-based approval.
- Consider a "simulate impact" mode before publishing auto-resolution rules.

### Account Mapping (`account_mapping`)

Account mapping rules confirm that external evidence belongs to an expected account or account family.

Currently supported `rule_type` values:

| Rule type | Purpose |
|---|---|
| `account_number_required` | Require an exact account number or account-number prefix. |

Config:

```json
{
  "external_account_field": "account_number",
  "account_match_mode": "prefix",
  "account_number": "011",
  "bank_name": "Access Bank"
}
```

Behavior:

- Exact mode requires the external account value to equal `account_number`.
- Prefix mode requires the external account value to start with `account_number`.
- If required account evidence is missing or mismatched, the engine raises `mapping_failure`.
- Direction is not configured in Account Mapping. Direction Check rules own direction mismatch validation.

Product guidance:

- Choose the external account field from the saved report mapping.
- Use `Exact account number` for one settlement account and `Account number starts with` for an account family.
- Settlement or GL account selection belongs to journal mapping, not reconciliation account matching.
- Missing mappings should produce actionable `mapping_failure` breaks.

## Configuration Principles

### Make Descriptions Required

The rule list should show human descriptions, not raw config. A good description explains:

- What the rule checks.
- Which source it applies to.
- What happens when it fails.
- Any tolerance or timing threshold.

Good:

> Allow up to NGN 10 variance for Paystack card payments to absorb processor rounding.

Bad:

> enabled: true

### Prefer Presets Over Blank JSON

Most operators should not write JSON. Product should provide presets:

- Paystack reference match
- Flutterwave status map
- GTBank narration extraction
- T+1 processor settlement window
- T+2 bank value date window
- Small amount tolerance
- Fee cap check

### Separate Finance-Owned And Ops-Owned Decisions

Finance-owned:

- Fee schedules
- Amount tolerances
- Auto-resolution thresholds
- Account mappings to GL context

Operations-owned:

- Break review
- Resolution notes
- Evidence upload/import
- Re-run workflows

Auditor-owned:

- Read-only review
- Evidence package downloads
- Rule version history inspection

## Product Decisions To Make

| Decision | Why it matters | Recommendation |
|---|---|---|
| Should every saved report source require a Reference Matching rule before import? | Prevents runs that cannot match anything. | Yes, require or show a blocking warning. |
| Should tolerance rules be global or report-source-specific? | Global tolerances are easy but can hide report-specific behavior. | Start report-source-specific, allow templates. |
| Should fuzzy matching be enabled? | It can reduce breaks but risks false positives. | Keep off until review UX is strong. |
| Should auto-resolution publish immediately? | Auto-closing breaks affects audit and financial control. | Require draft, simulation, and approval. |
| Should fee schedules be rules or separate objects? | Finance users need to see the fee logic where the rule is configured. | Store inline fee model fields on new rules; keep legacy schedule references readable. |
| Should rules apply to historical reruns? | Reruns can change historical outcomes if rules changed. | Store active rule versions on run artifacts. |
| Should Operations edit rules? | Rules alter financial evidence logic. | No. Admin/Finance only. |
| Should rule config remain JSON? | JSON is fast but not operator-friendly. | Replace with category-specific forms. |

## Current Rule Builder UX

The rule drawer uses category-specific forms:

- Reference Matching: internal field, external field, case sensitivity, prefix/suffix normalizers.
- Account Mapping: external account field, exact/prefix match mode, account number or prefix, optional bank name.
- Amount Tolerance: absolute tolerance, percentage tolerance, currency, report source scope.
- Fee Schedule: fee model, fee basis, percentage rate or flat fee, cap/minimum/waiver controls, fee field, net field, tolerance.
- Timing: Ledgerise transaction date, external settlement/value date field, allowed date difference, calendar/business day policy.
- Status Mapping: external values grouped by canonical status.
- Direction Check: internal direction, external direction field, expected external direction.
- Duplicate Detection: duplicate identity field, duplicate scope, optional reference window, optional amount-match requirement.
- Escalation: action, eligible break types, amount-difference limit, confidence threshold, suppression outcome explanation.

Every category form should generate:

- A human-readable description.
- A validated config object.
- An impact preview before publish.

## Implementation Gaps

Current engine support is solid for deterministic V1, but the product should not overpromise the advanced prototype behavior until these gaps close:

- Fuzzy matching is not currently implemented in the deterministic engine.
- Business-day timing windows exclude weekends but do not yet apply country-specific holiday calendars.
- Duplicate detection is identity-based, not full "same amount + reference + time window" logic.
- Auto-resolution category exists, but broad automatic closure policy needs workflow controls.
- Full report-header retention would let field dropdowns include unmapped columns, not only saved mapping columns.
- Legacy configs still need clear display paths when they use older fields or values.

## Legacy Config Notes

The drawer preserves older rule configs where possible:

- Account Mapping rules that previously used direction-style config should be recreated as Direction Check rules. Account Mapping now handles only account ownership or account-family checks.
- Existing account-number rules without `account_match_mode` are treated as exact account-number checks. Use prefix mode when the intent is an account family.
- Fee rules with `fee_schedule_ref` still display the legacy reference and can fall back to active legacy fee schedules. New net-after-fee rules should use inline fee model fields.
- Custom direction values and custom report field names still render through advanced custom controls.

## Decision Summary

For a commercially safe V1:

1. Require descriptions for all rules.
2. Keep only active rules in matching.
3. Make Reference Matching mandatory per saved report source.
4. Keep Amount Tolerance and Fee Schedule finance-controlled.
5. Keep Auto Resolution conservative and auditable.
6. Prefer source-specific presets over generic rules.
7. Add a rule test/preview step before publish.
8. Preserve version history and report which rules affected each run.
