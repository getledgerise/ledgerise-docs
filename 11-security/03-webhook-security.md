# webhook security

Inbound webhook endpoints are public URLs that receive financial data. Without payload verification, anyone who discovers the URL can send forged transactions into Ledgerise. This page explains how Ledgerise handles this and what you need to configure.

---

## how Ledgerise verifies webhook payloads

Every inbound webhook adapter verifies the **cryptographic signature** on each incoming payload before doing anything with it. Payloads that fail verification are rejected with an HTTP 401 response and are never normalized, stored, or surfaced in the Transactions page.

The verification flow:

1. Your source system computes an HMAC signature over the raw request body using the shared signing secret.
2. It includes the signature in a request header (the exact header name varies by provider — see below).
3. Ledgerise recomputes the HMAC using the signing secret you configured in Settings → Adapters.
4. If the signatures match, the payload is accepted and normalized. If they do not match, it is rejected.

---

## configuring your signing secret

Go to **Settings → Adapters → your webhook adapter → Configure** and enter the signing secret in the **Signing Secret** field. This must match exactly the secret configured in your source system.

If your source system generates the secret (for example, Paystack generates a webhook secret in its developer dashboard), copy it from there. If your internal payment system requires you to generate the secret, use a cryptographically random value of at least 32 bytes.

Signing secrets are stored encrypted. → See [data protection](02-data-protection.md#credentials-at-rest)

---

## provider-specific signature headers

Common payment providers send the HMAC signature in a specific header. Ledgerise's built-in adapters for these providers know which header to read:

| Provider | Signature header | Algorithm |
|---|---|---|
| Paystack | `x-paystack-signature` | HMAC-SHA512 |
| Flutterwave | `verif-hash` | Direct comparison (shared secret) |
| Generic webhook adapter | Configurable — set in adapter config | HMAC-SHA256 (default) |

For the generic webhook adapter, configure the signature header name and algorithm in **Settings → Adapters → webhook-adapter → Configure** to match what your source system sends.

---

## replay attack protection

Ledgerise rejects webhook payloads with a timestamp older than **5 minutes**. This protects against replay attacks: if an attacker captures a valid signed payload and tries to re-send it later, Ledgerise rejects it based on the timestamp, even though the signature is valid.

Your source system must include the current timestamp in the payload or in a dedicated timestamp header. The generic webhook adapter looks for a `timestamp` field in the payload body by default; configure an alternative path in the adapter config if your source system puts it elsewhere.

---

## rotating a signing secret

Rotate signing secrets periodically, or immediately if you suspect a secret has been exposed.

**The correct rotation order:**

1. Generate a new signing secret.
2. Enter the new secret in **Settings → Adapters → your adapter → Configure** and save.
3. Update your source system to sign payloads with the new secret.

Configure Ledgerise *first*, then your source system. Doing it the other way around creates a window where your source system is sending payloads signed with the new secret but Ledgerise is still expecting the old one — all webhooks during that window will be rejected and the transactions lost.

Rotation takes effect immediately on save. No restart or downtime is required.

---

## what happens to rejected payloads

Rejected payloads — those failing signature verification or timestamp validation — are:

- Responded to with an HTTP 401.
- Logged in the adapter error log with the rejection reason (signature mismatch or timestamp expired).
- Never stored or normalized.

Your source system's webhook delivery system will typically retry rejected payloads. If the rejection is due to a misconfigured secret, those retries will also fail until the secret is corrected. Check your source system's webhook delivery logs if you notice transactions from a specific time period are missing from Ledgerise.

---

## network-level controls

Signature verification is the primary defence. You can strengthen it with network-level controls:

**IP allowlisting** — if your source system publishes a fixed set of IP addresses for webhook delivery (Paystack, Flutterwave, and many providers do), configure your reverse proxy or firewall to only accept webhook traffic from those IPs on the webhook path. This prevents anyone outside the provider's network from even reaching the endpoint.

**Separate the webhook path from the dashboard** — configure your reverse proxy to route only the webhook path (`/adapters/webhook/*`) to the Ledgerise API, keeping the dashboard and API paths on a separate domain or requiring VPN access. This limits the public attack surface of your deployment.
