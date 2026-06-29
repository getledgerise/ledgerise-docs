# rule resolution order

When a transaction could match more than one rule, the engine needs a deterministic way to pick exactly one. This page explains that order, so you can write rules that behave the way you expect.

---

## the priority order

The engine evaluates rules in this order, highest priority first, and **stops at the first match**:

1. **Product line + biller (exact match)** — most specific. Use this for a single biller that needs different accounting treatment than the rest of its category.
2. **Product line + biller category** — use this for a group of billers in the same category that share the same accounts.
3. **Product line only (catch-all)** — use this as the default for everything on a product line that doesn't match a more specific rule.
4. **No match → suspense account** — if nothing matches, the transaction is posted to the configured suspense account and flagged as `unmapped`.

Lower-priority rules are never evaluated once a higher-priority rule matches. A transaction is never checked against more than one rule per run.

---

## a practical example

Suppose your `bill-payment` product line has two rules:

- A catch-all rule on `bill-payment` that credits a general **Bill Payment Revenue** account.
- A specific rule on `bill-payment` + biller `ikeja-electric` that credits a dedicated **Electricity Revenue** account.

A transaction for `ikeja-electric` matches the specific rule and posts to Electricity Revenue. A transaction for any other biller on the `bill-payment` product line falls through to the catch-all and posts to Bill Payment Revenue.

> **Start broad, then go specific.** Create a catch-all rule for each product line first. Add biller-specific rules only for the billers that genuinely need different accounting treatment — most billers in a category usually don't.

---

## matching a rule isn't the same as posting

Rule resolution determines *which accounts* a transaction will use — it runs independently of whether the transaction is actually allowed to post yet. If your deployment has a reconciliation posting gate configured to something other than `disabled`, a transaction can resolve to a rule and still wait for reconciliation before the journal entry is generated. → See [how Ledgerise works](../01-getting-started/how-ledgerise-works.md) for where reconciliation sits in the flow.

---

## seeing which rule was applied

To check which rule produced a specific journal entry:

1. Go to **Journal Log**.
2. Click the entry to open its detail drawer.
3. Look for **Mapping rule version applied** — it shows the exact rule version active at the time the entry was generated, not the rule's current version. For example: "Payments catch-all — v3, active from 12 Jan 2026." If the rule has since changed, a note flags that the current version differs.

This is what makes mapping rules auditable: even after you edit a rule, every journal entry it already produced still points to the version that was active when it posted.

→ See [rule audit trail](rule-audit-trail.md) for how rule versions are tracked
