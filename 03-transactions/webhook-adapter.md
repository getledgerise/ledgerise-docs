# webhook adapter

The webhook adapter receives transaction data pushed by your source system to a URL Ledgerise exposes. Each incoming request represents one transaction event. The adapter validates the payload, normalises it into the canonical format, and stores it immediately.

**Who configures this:** Admins and developers integrating a source system.

---

## how it works

1. You configure the webhook adapter with a shared signing secret and a field mapping.
2. Ledgerise provides you with a webhook endpoint URL — for example, `https://api.your-domain.com/webhooks/inbound`.
3. You configure your source system (payment engine, payment provider) to send POST requests to that URL whenever a transaction completes.
4. Each incoming payload is validated against the signing secret. Invalid signatures are rejected immediately with a `401` response.
5. Valid payloads are normalised into the canonical transaction schema and stored. The transaction appears on the Transactions page within seconds.

---

## configuration

Go to **Settings → Adapters → webhook-adapter → Configure**.

### endpoint url

Ledgerise shows you the webhook URL to give to your source system. Copy this and paste it into your source system's webhook configuration screen.

### signing secret

Enter a shared secret string. When your source system sends a webhook, it should include an `X-Ledgerise-Signature` header (or the header name you configure) containing an HMAC-SHA256 signature of the request body, signed with this secret.

Ledgerise verifies the signature on every incoming request. Payloads without a valid signature are rejected. This prevents unauthorised parties from posting fake transactions to your endpoint.

> Generate a strong signing secret using `openssl rand -hex 32` and configure it in both Ledgerise and your source system.

### field mapping

Your source system's payload probably uses different field names from the Ledgerise canonical schema. Field mapping tells the adapter where to find each canonical field in your payload.

For example, if your source system sends:

```json
{
  "tx_ref": "TXN-001-2024",
  "tx_amount": 500000,
  "tx_currency": "NGN",
  "product": "bill-payment",
  "biller_code": "ikeja-electric",
  "tx_status": "completed",
  "event_time": "2024-01-15T10:30:00Z"
}
```

You would map:
- `tx_ref` → `source_id`
- `tx_amount` → `amount`
- `tx_currency` → `currency`
- `product` → `product.line`
- `biller_code` → `product.biller`
- `tx_status` → `status`
- `event_time` → `occurred_at`

The field mapping UI presents your source field names on the left and canonical field names on the right. You drag or select the mapping for each required field.

[SCREENSHOT: Settings → Adapters → webhook-adapter configuration panel showing the endpoint URL, signing secret field, and the field mapping form with source fields on the left mapped to canonical fields on the right]

---

## enabling the adapter

After configuring the field mapping and signing secret, toggle the adapter to **Active**. Ledgerise runs a healthcheck — for the webhook adapter, this confirms the adapter configuration is valid and the endpoint is reachable.

The adapter tile in Settings → Adapters shows a green status badge when active.

---

## testing the webhook

Before going live with real traffic, send a test payload from your source system to the webhook URL and confirm the transaction appears on the Transactions page.

If your source system has a built-in webhook test function, use it. Otherwise, you can send a test payload directly using curl:

```bash
# Generate the signature (use your actual secret)
SECRET="your-signing-secret"
PAYLOAD='{"tx_ref":"TEST-001","tx_amount":100000,"tx_currency":"NGN","product":"bill-payment","tx_status":"completed","event_time":"2024-01-15T10:00:00Z"}'
SIGNATURE=$(echo -n "$PAYLOAD" | openssl dgst -sha256 -hmac "$SECRET" | awk '{print $2}')

curl -X POST https://api.your-domain.com/webhooks/inbound \
  -H "Content-Type: application/json" \
  -H "X-Ledgerise-Signature: $SIGNATURE" \
  -d "$PAYLOAD"
```

Go to **Transactions** and look for your test record. It should appear with a posting status of `unposted`.

[SCREENSHOT: Transactions page showing a newly ingested webhook transaction with source reference TEST-001, status completed, and posting status unposted]

---

## what happens if a webhook payload fails

Ledgerise responds to every incoming webhook request within 200ms — success or failure. Normalisation happens asynchronously after the response is sent. This means your source system never waits for Ledgerise to finish processing before getting a confirmation.

If a payload fails validation — for example, a required field is missing or the date format is unrecognised — the error is recorded in the adapter log. Malformed records do not appear on the Transactions page as normal records; they appear as ingestion errors in the adapter detail view under Settings → Adapters → webhook-adapter.

If your source system retries failed webhooks (most do), valid retries are accepted normally. Duplicates are detected by `source_id` and skipped — so retries are safe.

---

## for provider-specific integrations

The generic webhook adapter uses field mapping to handle any payload format. For payment providers with their own specific formats — Paystack, Flutterwave, and others — provider-specific adapters can be built and registered. These follow the same interface but handle format-specific quirks internally, removing the need for manual field mapping.

→ See [building an adapter](../08-adapters/building-an-adapter.md) for the adapter interface specification.
