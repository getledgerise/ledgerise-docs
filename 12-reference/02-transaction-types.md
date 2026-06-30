# transaction types

The `type` field on every canonical transaction record uses dot notation: `category.subcategory` or `category.subcategory.detail`. There are 84 standard types across 11 categories. Custom types are permitted when a matching mapping rule exists.

The engine uses `type` together with `product.line`, `product.biller`, and `product.biller_category` to resolve the correct COA accounts. An unrecognised type without a mapping rule routes the transaction to the suspense account.

---

## payment

Payment transactions represent a customer paying a biller or service provider through the operator's platform.

| Type | Description |
|---|---|
| `payment.airtime` | Mobile airtime top-up for a subscriber. |
| `payment.data` | Mobile data bundle purchase. |
| `payment.electricity` | Electricity distribution company token or prepaid meter payment. |
| `payment.water` | Water utility bill payment. |
| `payment.cable-tv` | Pay-TV subscription payment (e.g. DStv, GOtv, StarTimes). |
| `payment.internet` | Broadband or ISP subscription payment. |
| `payment.insurance-premium` | Insurance premium payment (life, health, auto, or other). |
| `payment.government-levy` | Government tax, levy, or regulatory fee payment. |
| `payment.education` | School fees or other education-related payment. |
| `payment.transport` | Transport fare, logistics, or mobility payment. |
| `payment.betting` | Sports betting stake or lottery entry. |
| `payment.merchant` | General payment to a merchant not covered by a more specific type. |
| `payment.invoice` | Payment against a specific invoice. |
| `payment.subscription` | Recurring subscription payment (SaaS, streaming, membership). |

---

## transfer

Transfer transactions represent the movement of funds between wallets, accounts, or banking channels.

| Type | Description |
|---|---|
| `transfer.wallet-to-wallet` | Transfer between two wallets within the operator's platform. |
| `transfer.wallet-to-bank` | Withdrawal from a wallet to an external bank account. |
| `transfer.bank-to-wallet` | Funding of a wallet from a bank account. |
| `transfer.agent-to-wallet` | Cash-in via an agent, credited to a customer wallet. |
| `transfer.wallet-to-agent` | Cash-out from a wallet via an agent. |
| `transfer.internal` | Internal ledger movement between accounts within the operator (no net cash movement). |

---

## collection

Collection transactions represent funds received by the operator from a customer or counterparty via a specific channel.

| Type | Description |
|---|---|
| `collection.pos` | Payment collected via a POS terminal. |
| `collection.web` | Payment collected via a web checkout or payment page. |
| `collection.mobile` | Payment collected via a mobile app. |
| `collection.ussd` | Payment collected via a USSD session. |
| `collection.qr` | Payment collected via QR code scan. |
| `collection.nfc` | Payment collected via NFC tap. |
| `collection.api` | Payment collected via a direct API integration with a corporate or business customer. |
| `collection.agent` | Cash collected by an agent on behalf of the operator. |
| `collection.bank-transfer` | Payment received via a bank transfer or NIP/NEFT instruction. |
| `collection.direct-debit` | Payment collected via a direct debit mandate. |

---

## fee

Fee transactions represent charges applied by or to the operator in connection with a transaction or service.

| Type | Description |
|---|---|
| `fee.platform` | Fee charged to the customer by the operator. Operator revenue. |
| `fee.processing` | Fee charged to the operator by an aggregator or processor. Operator cost. |
| `fee.withdrawal` | Fee charged on a wallet-to-bank withdrawal. |
| `fee.maintenance` | Periodic account maintenance fee. |
| `fee.card` | Card issuance, annual, or management fee. |
| `fee.fx` | Foreign exchange conversion fee. |
| `fee.late-payment` | Penalty fee applied for a late payment. |
| `fee.reversal` | Fee charged in connection with a transaction reversal. |
| `fee.chargeback` | Chargeback penalty fee. |

---

## loan

Loan transactions represent disbursements, repayments, and adjustments on lending products.

| Type | Description |
|---|---|
| `loan.disbursement` | Loan principal paid out to the borrower. |
| `loan.repayment.principal` | Repayment of loan principal. |
| `loan.repayment.interest` | Repayment of accrued interest. |
| `loan.repayment.penalty` | Repayment of penalty charges on a loan. |
| `loan.repayment.fee` | Repayment of loan origination or processing fees. |
| `loan.write-off` | Write-off of an uncollectable loan balance. |
| `loan.provision` | Provision created for expected credit loss on a loan. |
| `loan.restructure` | Entry recording a loan restructuring or rescheduling. |

---

## savings

Savings transactions represent deposits, withdrawals, and returns on savings products.

| Type | Description |
|---|---|
| `savings.deposit` | Customer deposits funds into a savings account or product. |
| `savings.withdrawal` | Customer withdraws from a savings account. |
| `savings.interest-credit` | Interest credited to a savings balance. |
| `savings.liquidation` | Full savings account closure and payout of balance. |

---

## investment

Investment transactions represent purchases, maturities, and returns on investment products.

| Type | Description |
|---|---|
| `investment.purchase` | Customer purchases an investment product (e.g. treasury bills, mutual fund units). |
| `investment.maturity` | Payout of an investment at its maturity date. |
| `investment.yield-payout` | Periodic yield or dividend distribution to the customer. |
| `investment.liquidation` | Early or full liquidation of an investment before maturity. |

---

## remittance and fx

Remittance and foreign exchange transactions. The type prefix matches the nature of the transaction: `remittance.*` for cross-border money transfers, `fx.*` for currency conversion and associated entries.

| Type | Description |
|---|---|
| `remittance.send` | Outbound remittance — funds sent abroad or to another country. |
| `remittance.receive` | Inbound remittance — funds received from abroad. |
| `remittance.fee` | Fee charged on a remittance transaction. |
| `fx.conversion` | Foreign exchange conversion of one currency to another. |
| `fx.fee` | Fee charged on an FX conversion. |
| `fx.gain` | Realized gain from an FX conversion or position. |
| `fx.loss` | Realized loss from an FX conversion or position. |

---

## card

Card transactions represent activity on prepaid, debit, or virtual card products issued by the operator.

| Type | Description |
|---|---|
| `card.load` | Funds loaded onto a card from a wallet or bank account. |
| `card.spend` | Card spend at a merchant or POS. |
| `card.reversal` | Reversal of a card spend transaction. |
| `card.chargeback` | Chargeback initiated by the cardholder. |
| `card.chargeback-reversal` | Reversal of a previously accepted chargeback. |
| `card.fee` | Card-related fee (issuance, annual, management). |
| `card.expiry-credit` | Credit of unused balance back to the customer on card expiry. |

---

## agency

Agency banking transactions represent cash and float movements through the agent network.

| Type | Description |
|---|---|
| `agency.cash-in` | Customer deposits cash at an agent location, credited to their wallet. |
| `agency.cash-out` | Customer withdraws cash at an agent location, debited from their wallet. |
| `agency.commission` | Commission earned by an agent for a cash-in, cash-out, or other transaction. |
| `agency.vault-deposit` | Cash deposited by the operator into an agent's physical vault. |
| `agency.vault-withdrawal` | Cash withdrawn from an agent's vault by the operator. |
| `agency.float-allocation` | Float value allocated from the operator to an agent to fund operations. |
| `agency.float-recovery` | Float recovered from an agent back to the operator. |

---

## system

System transactions are generated internally by Ledgerise or the operator for ledger management and do not correspond to a customer-facing event.

| Type | Description |
|---|---|
| `system.reversal` | System-generated reversal entry for an earlier transaction. |
| `system.refund` | System-generated refund entry. |
| `system.adjustment` | Manual or automated ledger adjustment entry. |
| `system.settlement-batch` | Entry representing a net settlement batch across multiple transactions. |
| `system.suspense-debit` | Debit entry posted to the suspense account pending resolution. |
| `system.suspense-credit` | Credit entry posted to the suspense account pending resolution. |
| `system.opening-balance` | Opening balance entry for a new period or account. |
| `system.closing-balance` | Closing balance entry for a period or account. |

---

## custom types

If your platform has a transaction type not covered by the standard taxonomy, you can define a custom type. Custom types must:

1. Follow the same dot notation as standard types — `category.subcategory` or `category.subcategory.detail`.
2. Have a corresponding mapping rule. A custom type with no matching rule routes to the suspense account like any other unmatched transaction.
3. Be documented in the adapter README that emits them.

Custom types do not require schema changes. The engine treats any `type` value not in the standard taxonomy as a custom type and applies the normal rule resolution order.
