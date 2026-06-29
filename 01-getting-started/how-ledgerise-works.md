# how ledgerise works

This page explains the architecture and the data flow inside Ledgerise. You do not need to understand all of this to use the product day to day, but it is helpful context before you start configuring adapters and mapping rules.

---

## the three layers

Ledgerise is built in three layers. Each layer has a single, clear responsibility. They are kept deliberately separate so that adding a new payment provider or a new accounting system requires only a new adapter — the logic in the middle never changes.

```
Source System              Ledgerise Core                    Accounting System
─────────────              ──────────────                    ─────────────────

Webhook JSON    ──►                                         
CSV Upload      ──►   Inbound Adapters   ──►   Journal   ──►   Zoho Books
API Poll        ──►   (normalize data)         Engine    ──►   Journal CSV
Any System      ──►                         (map + post) ──►   QuickBooks
                              │
                              ▼
                       Internal Database
                       (transactions, rules,
                        journal entries)
```

[SCREENSHOT: Ledgerise system architecture diagram showing inbound adapters on the left feeding into the journal engine, which sends output to outbound adapters on the right, with the internal database shown below the engine]

---

### inbound adapters

An inbound adapter is a module that knows how to speak to one specific source system and translate its transaction data into the standard format Ledgerise uses internally.

Your payment engine might call a completed electricity payment `tx_completed_electricity`. Paystack calls it differently. Flutterwave calls it something else again. The adapter's job is to normalise all of these into one consistent format — the canonical transaction schema — so that everything downstream works the same way regardless of where the data came from.

Ledgerise includes three general-purpose inbound adapters out of the box:

| Adapter | How it receives data |
|---|---|
| Webhook | Your source system pushes individual transaction events to a URL Ledgerise exposes |
| CSV Import | You upload a flat file exported from your source system |
| Poll | Ledgerise calls your source system's API on a schedule and fetches new transactions |

For payment providers with their own specific formats — Paystack, Flutterwave, M-Pesa, and others — provider-specific adapters can be built and registered. They follow the same rules and produce the same output. Only the translation logic differs.

---

### the canonical transaction schema

The canonical schema is the internal language Ledgerise speaks. Every inbound adapter, regardless of source system, must produce a transaction record in this format before anything else can happen.

The schema captures the essentials of every transaction:

- What happened and when (`type`, `occurred_at`, `completed_at`)
- How much, in what currency, and in which direction (`amount`, `currency`, `direction`)
- Which product line, biller, and channel it belongs to (`product.line`, `product.biller`, `channel`)
- The transaction's current status (`status`)
- Where it came from (`source.adapter`, `source.system`)

Why does this matter? Because once a transaction is in the canonical schema, the journal engine does not need to know it came from Paystack or from a CSV export. It sees a completed airtime payment on the consumer-app product line, matches it to the right mapping rule, and posts the journal entry. The mapping rules, engine logic, and outbound adapters are all written once, for one schema.

→ See [transaction schema reference](../12-reference/transaction-schema.md)

---

### the journal engine

The journal engine is the core of Ledgerise. It runs on a configurable schedule — the default is every hour — and does the following in sequence:

1. Fetch all completed, unposted transactions from the database.
2. Deduplicate: skip any transaction whose source ID has already been posted.
3. For each transaction, find the matching mapping rule.
4. Generate a double-entry journal entry — a debit and a credit that balance exactly.
5. Send the journal entries to the outbound adapter in batches.
6. Record the result for each entry: posted, failed, or unmapped.

The engine never guesses. If it cannot match a transaction to a mapping rule, it does not drop the record or invent an account. It posts the transaction to your configured suspense account and flags it as unmapped so your finance team can review it.

---

### mapping rules

Mapping rules are the configuration that tells the engine which accounts to debit and credit for a given transaction. They are created and managed by your finance team through the Ledgerise UI — no code changes are required.

A rule might say: for any completed transaction on the `bill-payment` product line with biller `ikeja-electric`, debit `Accounts Receivable` and credit `Electricity Revenue`.

Rules are evaluated in priority order:

1. Exact match on product line + biller — most specific, highest priority
2. Match on product line + biller category
3. Match on product line only — a catch-all for that product line
4. No match → suspense account

This means you can define a default rule for an entire product line, and then override it with a more specific rule for individual billers that need different accounting treatment.

→ See [mapping rules overview](../05-mapping-rules/overview.md)

---

### outbound adapters

An outbound adapter takes the journal entries the engine has generated and delivers them to your accounting system.

Ledgerise ships with two outbound adapters:

| Adapter | What it does |
|---|---|
| Zoho Books | Posts journal entries directly to Zoho Books via the Zoho API |
| Journal CSV | Exports journal entries as a CSV file for manual import into any accounting system |

The Zoho Books adapter handles authentication, batching, rate limiting, and automatic retry. If your accounting system is not yet supported by an API adapter, the journal CSV adapter gives you a lowest-friction path to get your entries out.

---

## key behaviours to understand

### deduplication

Ledgerise prevents the same transaction from being posted twice. Every inbound adapter records a `source_id` — the stable unique identifier from the original system, such as a Paystack transaction reference. Before posting, the engine checks whether that `source_id` has already been processed. Duplicates are marked and skipped, never posted.

### suspense

No transaction is ever silently dropped or misposted. If the engine cannot find a matching mapping rule, it posts the transaction to a suspense account — a designated holding account in your chart of accounts — and flags it as unmapped. Your finance team sees it in the Transactions page and can assign the correct rule when they are ready.

This is a deliberate design choice. A misposting is far harder to unwind than an unmapped transaction waiting in suspense.

### test environment isolation

Many payment systems have a test or sandbox environment. Transactions from a test environment are stored in Ledgerise normally, but the journal engine will never post them to your accounting system. They are permanently excluded from posting. This means you can safely connect a test payment integration without any risk of polluting your books.

### reversal handling

When a transaction is reversed after its journal entry has already been posted, Ledgerise generates a mirror journal entry — the same amounts, but with debits and credits swapped — dated to the reversal date. If the original transaction had not yet been posted when the reversal arrived, it is simply cancelled. Nothing is ever left in an inconsistent state.

---

## what happens when something goes wrong

Failed postings — for example, if Zoho Books is temporarily unreachable — are retried automatically with increasing wait times: 5 minutes, 15 minutes, 1 hour, 4 hours, and 24 hours. After five failed attempts, the entry is flagged as needing manual attention and appears prominently in the Journal Log.

Adapter errors — for example, a malformed CSV or a webhook payload that fails validation — are recorded with a clear error message so you can diagnose and fix the underlying problem without losing the transaction.

---

## next steps

Now that you understand how Ledgerise is structured, you are ready to start using it.

- If you are setting up Ledgerise for the first time, follow the [quickstart](quickstart.md).
- If you are looking for definitions of specific terms, see [key concepts](key-concepts.md).
- For full deployment instructions, see [deployment overview](../02-deployment/overview.md).
