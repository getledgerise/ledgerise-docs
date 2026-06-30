# error codes

Standard error codes returned by adapters in failure envelopes. These codes appear in the `code` field of the failure envelope and in adapter error logs visible in the Journal Log and Settings → Adapters.

```json
{
  "status": "error",
  "code": "AUTH_FAILED",
  "message": "Paystack rejected the secret key — check for expiry or rotation",
  "raw": { ... }
}
```

---

## standard codes

| Code | Description | Common cause | Suggested action |
|---|---|---|---|
| `VALIDATION_FAILED` | Input failed schema validation before normalization. | A required field is missing or the wrong type. | Check the `errors` array in the envelope for field-level detail. Fix the source payload or adapter mapping. |
| `SOURCE_UNREACHABLE` | The adapter could not connect to the source system. | Network issue, firewall rule, or provider downtime. | Run the adapter healthcheck in Settings → Adapters. Confirm network access from your server to the provider. |
| `AUTH_FAILED` | The source system rejected the credentials the adapter presented. | Expired API key, rotated secret, or incorrect credentials. | Re-enter credentials in Settings → Adapters → your adapter → Configure. |
| `RATE_LIMITED` | The source system returned a rate limit response. | Poll adapter calling too frequently, or burst of webhook deliveries. | Ledgerise retries automatically on a backoff schedule. If the error recurs, reduce the poll frequency in adapter config. |
| `MALFORMED_PAYLOAD` | The input could not be parsed at all. | Invalid JSON, corrupt file, wrong encoding, or truncated payload. | Check the raw input in the error log. Verify the source system's encoding settings and payload format. |
| `UNSUPPORTED_EVENT` | The event type from the source system is not handled by this adapter. | The provider added a new event type the adapter does not map. | Update the adapter to handle the new type, or add a mapping in the adapter code. |
| `METHOD_NOT_SUPPORTED` | The called method is not supported by this adapter's declared modes. | Calling `normalize()` on a webhook adapter in poll mode, or vice versa. | Use an adapter that supports the required mode, or check the adapter's `meta().modes` declaration. |
| `ADAPTER_*` | A custom adapter-specific error. The suffix is defined by the adapter author. | Varies — specific to the adapter's source system or internal logic. | See the adapter's README for documentation of its custom error codes. |

---

## where error codes appear

**Journal Log** — the retry drawer shows the error code and message for each failed posting attempt. Use this for `AUTH_FAILED`, `RATE_LIMITED`, and `SOURCE_UNREACHABLE` errors during outbound posting.

**Settings → Adapters** — the adapter list shows the last healthcheck status. A failed healthcheck surfaces `SOURCE_UNREACHABLE` or `AUTH_FAILED` codes from the `healthcheck()` method.

**Adapter error log** — available from the adapter detail drawer in Settings → Adapters. Shows the full failure envelope including `code`, `message`, and `raw` input for inbound normalization failures.

---

## custom codes

Adapters may define custom error codes using the `ADAPTER_` prefix — for example, `ADAPTER_INVALID_ACCOUNT_TYPE` or `ADAPTER_MISSING_FLOAT_REF`. Custom codes must be documented in the adapter's README. They appear in the same failure envelope format as standard codes.
