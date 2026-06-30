# unmapped transactions

An unmapped transaction is one the journal engine processed but could not match to a mapping rule. Instead of dropping the record or posting to a wrong account, Ledgerise posts it to a designated **suspense account** and flags it as `unmapped`. Your finance team reviews and resolves these.

---

## why transactions become unmapped

When the engine processes a transaction, it looks for a mapping rule that matches the transaction's product line, biller, and biller category — in that order, from most specific to least specific. If no matching rule exists at any level, the transaction goes to the suspense account.

The most common reasons:

- **A new biller** was added to your platform and no rule has been created for it yet.
- **A new product line** was introduced and the finance team has not configured rules for it.
- **The biller identifier** in the source data does not match what is configured in your mapping rules — for example, `ikeja-electric-ltd` when the rule expects `ikeja-electric`.
- **Initial setup** — when you first go live, it is normal for some transactions to be unmapped until your rule coverage is complete.

The suspense account is a safety net. No transaction is ever silently dropped or posted to a wrong account.

---

## how to find unmapped transactions

**From the stat bar** — the Transactions page stat bar shows an unmapped count. A non-zero number means transactions are waiting in suspense.

**From the filter bar** — on the Transactions page, click the **Unmapped** quick filter (or set the posting status filter to `unmapped`). This shows all transactions currently in the suspense account.

**From the exceptions badge** — the Exceptions badge in the top navigation bar aggregates all open issues, including unmapped transactions. Clicking it shows a filtered view.

[SCREENSHOT: Transactions page filtered to "Unmapped only" showing amber posting status badges on each row, with an "Assign Rule" action button visible on a highlighted row]

---

## how to resolve an unmapped transaction

Each unmapped transaction has an **Assign Rule** action button in the row and in the detail drawer.

Clicking **Assign Rule** opens the Mapping Rules page, pre-filtered to the product line and biller of that transaction. From there you can:

- **Create a new rule** — if no rule exists for this combination, click **Add Rule** and configure the debit and credit accounts for this product line and biller.
- **Find an existing rule** — if a rule already exists but the biller identifier does not match, check whether the biller value on the transaction matches the biller value in the rule. A mismatch in casing or spelling will prevent the rule from matching.

→ Step-by-step: [creating a rule](../05-mapping-rules/02-creating-a-rule.md)

---

## what happens after you create the rule

Once a new mapping rule is active, the engine will pick up the previously unmapped transactions and process them on the next run.

You do not need to do anything manually — the engine re-evaluates all `unmapped` transactions automatically. On the next engine run, the matching transactions will move from `unmapped` to `posted` (or `failed` if the accounting system returns an error).

You can trigger a manual engine run from **Journal Log → Run Engine Now** if you do not want to wait for the next scheduled run.

---

## how many unmapped transactions is normal

| Period | Expected unmapped rate |
|---|---|
| First week after go-live | Can be high — depends on how many product lines and billers you covered during setup |
| After initial rule coverage is complete | Below 2% of daily transaction volume |
| Steady state | Below 0.5% — occasional new billers or product lines only |

If your unmapped rate is consistently above 2%, your mapping rule coverage is incomplete. Use the **Unmapped only** filter to see which product lines and billers are most commonly unmapped, and create rules for them.

---

## the suspense account on your books

All unmapped transactions are posted to the suspense account you configured in Settings → System. This account should exist in your chart of accounts as a liability or clearing account.

On your accounting system side, the suspense account balance represents the total value of transactions Ledgerise has ingested but not yet correctly classified. As you create mapping rules and the engine re-posts unmapped transactions, the suspense balance decreases. A well-configured Ledgerise deployment maintains a near-zero suspense balance in steady state.

---

## investigating a specific unmapped transaction

Click any unmapped row to open the detail drawer. It shows:

- The full canonical transaction record, including `product.line`, `product.biller`, and `product.biller_category` values — these are the exact values the engine used when looking for a matching rule.
- The reason the transaction was not matched (no rule found for this combination).
- The suspense account the transaction was posted to.
- A direct link to create a mapping rule for this combination.

Use the `product.line` and `product.biller` values from the drawer when creating your rule, to make sure the new rule will match future transactions with the same values.
