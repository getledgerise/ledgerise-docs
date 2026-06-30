# data protection

This page covers how Ledgerise protects data at rest, in transit, and at the boundaries of your deployment.

---

## credentials at rest

Adapter credentials, API keys, and secrets stored in Ledgerise — including Zoho Books OAuth tokens, webhook signing secrets, poll adapter API keys, and AI provider keys — are encrypted at rest using **AES-256-GCM**.

The encryption key is your `LEDGERISE_CREDENTIALS_KEY` environment variable: a 64-character hex value you generate and control. Ledgerise never has access to this key and cannot decrypt your credentials on your behalf.

**This key must never be stored in source control.** If your `LEDGERISE_CREDENTIALS_KEY` is committed to a repository:

1. Rotate all credentials stored in Ledgerise immediately — they should be considered compromised.
2. Generate a new `LEDGERISE_CREDENTIALS_KEY`.
3. Re-enter all credentials in Settings → Adapters.

Store the key in a secrets manager (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager) or as a protected environment variable in your deployment platform.

---

## PII handling

Ledgerise applies masking at the inbound adapter layer. By the time a transaction record reaches the database, sensitive values have been reduced to their non-sensitive form:

| Data type | What is stored |
|---|---|
| Phone numbers | Last 4 digits only — for example, `****5678` |
| Card numbers | Last 4 digits only — for example, `****1234` |
| BVN (Bank Verification Number) | Never stored |
| NIN (National Identification Number) | Never stored |

This masking is enforced by the adapter specification (normalize rule 6) and cannot be bypassed through the UI. Custom adapters that emit unmasked PII violate the spec and should not be registered.

Full transaction amounts and account codes are stored — these are necessary for posting accurate journal entries and for reconciliation. They are not considered PII under the spec, but they are financial data and should be treated as sensitive for access control and backup encryption purposes.

---

## data in transit

**Dashboard and API traffic** must be served over HTTPS with TLS 1.2 or higher. Configure TLS termination at your reverse proxy — nginx, Caddy, or a cloud load balancer. The Ledgerise API and dashboard do not handle TLS directly; they expect to be placed behind a TLS-terminating proxy.

Do not expose the Ledgerise API (`localhost:3000`) or dashboard (`localhost:3001`) directly to the internet without TLS. The deployment guides configure nginx to handle this.

**Database connections** should use SSL/TLS between the Ledgerise API and your PostgreSQL database. Set `?sslmode=require` (or `verify-full` for stricter validation) in your `DATABASE_URL`. Most managed PostgreSQL providers enforce encrypted connections by default.

---

## AI provider keys

If you use Ledgerise's natural language features (the natural language report mode in Reconciliation), you must configure an AI provider API key in **Settings → System → AI Provider**. This key is stored encrypted alongside your adapter credentials using the same AES-256-GCM scheme.

Ledgerise uses your AI key exclusively within your deployment to process natural language queries you initiate. The key is not shared with Ledgerise and is not used outside your deployment.

---

## what leaves your deployment

Ledgerise sends one type of data outside your deployment: **summary-level usage counts** during the daily license check-in. These contain only the number of transactions ingested in the current calendar month — no transaction IDs, amounts, account codes, customer identifiers, or financial records of any kind are transmitted.

No other data leaves your deployment. Ledgerise does not have telemetry, crash reporting, or analytics that phone home.

---

## environment variable security

Several environment variables in Ledgerise contain sensitive material:

| Variable | Sensitivity | Risk if exposed |
|---|---|---|
| `DATABASE_URL` | Critical | Full database access |
| `LEDGERISE_CREDENTIALS_KEY` | Critical | Decrypts all stored adapter credentials |
| `AUTH_TOKEN_SECRET` | Critical | Allows forging session tokens |
| `AI_API_KEY` (if set) | High | Unauthorized AI API usage billed to your account |

These variables must not appear in:
- Source control (`.env` files committed to a repo)
- Application logs
- Error messages surfaced to end users
- Chat messages or support tickets

Use a secrets manager or your deployment platform's protected environment variable mechanism for all of these.
