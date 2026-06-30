# generic webhook adapter

The `webhook-adapter` lets you connect any payment system or internal service that can send JSON event data to a URL. You configure a field mapping in the Ledgerise UI that tells the adapter how to find each canonical field in your payload — no code required.

---

## when to use this adapter

Use the generic webhook adapter when:

- Your source system can POST transaction events to a URL on each transaction.
- Your payload is JSON (single record or array of records).
- The field mapping from your payload to the canonical schema is straightforward — no conditional logic, computed fields, or proprietary encoding.

If your source system requires a custom authentication handshake, has a non-JSON format, or needs conditional type mapping logic, consider [building a custom adapter](03-building-an-adapter.md) instead.

---

## how it works

1. Ledgerise exposes a unique webhook URL for your deployment.
2. You configure your source system to POST transaction events to that URL.
3. The adapter receives the payload, runs it through your field mapping, validates the result, and normalizes it into the canonical transaction schema.
4. The transaction record appears in the Transactions page within seconds.

---

## configuration

Go to **Settings → Adapters → webhook-adapter → Configure**.

| Field | Required | Description |
|---|---|---|
| Endpoint path | Yes | The URL path Ledgerise exposes for this webhook. Copy this and enter it in your source system. |
| Signing secret | No | A shared secret used to verify the payload's signature. If your source system supports HMAC signature verification, enter the secret here. Ledgerise will reject payloads that fail signature verification. |
| Field mapping | Yes | Maps canonical schema fields to paths in your JSON payload (dot notation supported). |

---

## field mapping

The field mapping tells the adapter where to find each canonical field in your payload. Use dot notation for nested fields.

Example: if your payload looks like this:

```json
{
  "txn": {
    "id": "PAY-20260601-00123",
    "amount_ngn": 5000,
    "status": "successful",
    "type": "bill_payment",
    "timestamp": "2026-06-01T14:23:00Z"
  },
  "customer": {
    "phone": "08012345678"
  }
}
```

Your mapping would be:

| Canonical field | Source path |
|---|---|
| `source_id` | `txn.id` |
| `amount` | `txn.amount_ngn` |
| `status` | `txn.status` |
| `type` | `txn.type` |
| `occurred_at` | `txn.timestamp` |
| `principal.reference` | `customer.phone` |

---

## supported payload formats

The adapter accepts JSON payloads containing either:

- **A single record** — the top-level object is the transaction.
- **An array of records** — the top-level object is an array; each element is processed as a separate transaction.

XML, form-encoded, and other non-JSON formats are not supported by the generic adapter. Build a [custom adapter](03-building-an-adapter.md) for those.

---

## testing your webhook

Once configured, send a test payload from your source system to the webhook URL. The transaction should appear in the Transactions page within a few seconds. If it doesn't:

1. Check **Settings → Adapters → webhook-adapter** for a red healthcheck status or error notice.
2. Go to the Transactions page and look for a record with `posting_status: failed` — the detail drawer will show the validation error from the adapter.
3. Confirm your payload matches the expected structure and that your field mapping paths are correct.

---

## limitations

- JSON only. No XML, CSV, or proprietary binary formats.
- No built-in support for pagination or bulk payloads beyond a top-level array.
- No conditional field mapping — if a canonical field needs to be derived from multiple source fields or requires conditional logic, build a custom adapter.
- Status mapping: the adapter attempts to map common status strings (`successful`, `failed`, `pending`, `reversed`, `disputed`) to canonical values. If your source system uses non-standard status values, you may need a custom adapter.
