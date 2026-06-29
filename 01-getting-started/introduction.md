# introduction

Ledgerise is a payment operations infrastructure layer. It sits between your transaction system and your accounting software, and its job is simple: take every completed payment transaction your platform processes and turn it into an accurate, double-entry journal entry — automatically, without anyone on your team doing it by hand.

---

## the problem

Payment operators in emerging markets typically process thousands of transactions a day across multiple product lines, billers, and channels. Their transaction systems record every event in detail. But that operational data does not automatically flow into their accounting system.

The result is a gap. Revenue is not recorded at the transaction level in the general ledger. Float balances do not appear as assets on the balance sheet. Customer wallet liabilities go untracked. Every month, the finance team spends significant time manually compiling figures, reconciling records, and correcting mispostings — work that does not scale, and is prone to error.

This is not a niche problem. It affects almost every fintech operator that has grown beyond a handful of transactions per day.

---

## what ledgerise does

Ledgerise closes that gap. It receives your transaction data, verifies it against external counterparty records, classifies each transaction using rules your finance team configures, and posts a correct double-entry journal entry to your accounting system — all without engineering involvement once the initial setup is complete.

The core loop looks like this:

```
Transactions in  →  Reconciliation  →  Classification  →  Journal entries out
```

1. **Transactions in** — your payment data reaches Ledgerise via webhook, CSV upload, or scheduled API poll.
2. **Reconciliation** — Ledgerise compares your internal records against external statements from your payment providers and banks to verify that what you recorded matches what they recorded.
3. **Classification** — your mapping rules tell Ledgerise which accounting accounts to debit and credit for each transaction type.
4. **Journal entries out** — Ledgerise posts the completed entries to your accounting system (such as Zoho Books) or exports them as a CSV you can import manually.

[SCREENSHOT: Ledgerise dashboard showing the top navigation bar with Transactions, Reconciliation, Mapping Rules, Journal Log, and Settings tabs]

---

## what ledgerise is not

Understanding the boundaries of the product helps you set the right expectations before getting started.

**Ledgerise is not an accounting system.** It does not store a general ledger, produce financial statements, or manage your chart of accounts. It posts entries into your existing accounting system — you manage that system separately.

**Ledgerise is not a transaction monitoring system.** It does not originate transaction data. It consumes data from the system that does — your payment engine, gateway, or POS network.

**Ledgerise is not a replacement for your accountant.** It automates the mechanical posting work. Judgment calls, period-end adjustments, tax considerations, and financial analysis remain human responsibilities.

**Ledgerise does not handle FX conversion.** It records the currency on each transaction and passes it to your accounting system. Currency conversion, if needed, is handled there.

---

## who uses ledgerise

Ledgerise is designed for three types of users within a fintech operator's team.

**Finance Officer / Accountant** — The primary day-to-day user. Configures and manages mapping rules, monitors the journal log, reconciles external statements, and reviews any transactions that need manual attention. Does not write code.

**Admin** — Handles the initial deployment, connects adapters to source systems and the accounting system, manages user accounts, and configures system settings. This is typically an engineer or DevOps team member.

**Auditor** — Has read-only access to the journal log, transactions, and audit trail. Typically an internal or external auditor who needs to trace postings back to source transactions.

---

## where to go next

If you want to understand how Ledgerise works before touching anything, read [how ledgerise works](how-ledgerise-works.md).

If you are the person setting up Ledgerise for your organisation and want to get started quickly, go to the [quickstart](quickstart.md).

If you have encountered a term in this documentation that you are not familiar with, the [key concepts](key-concepts.md) page explains the vocabulary used throughout.
