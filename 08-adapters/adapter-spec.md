# adapter specification

This is the technical contract every Ledgerise adapter must fulfil. All adapters — built-in and custom — implement the same interface so the engine can discover, configure, and invoke them uniformly.

**Spec version: 1.0.0**

---

## what an adapter is

An adapter is a self-contained module that knows how to speak to one specific source or destination system and translate between that system's format and Ledgerise's internal canonical schema. The boundary is strict: adapters handle translation, nothing else. No journal logic. No COA account references. No direct writes to accounting systems.

The engine knows nothing about external systems. Adapters are the only thing that does. Keeping these concerns separate is what allows the ecosystem to grow without touching the engine.

---

## adapter modes

Every adapter must declare which of these modes it supports. An adapter may support more than one.

| Mode | How the adapter receives input |
|---|---|
| `webhook` | Raw HTTP request body and headers pushed by the source system |
| `poll` | A cursor object — `{ last_fetched_at, last_source_id }` — passed by the engine on each scheduled invocation |
| `file_import` | A file buffer and metadata — `{ filename, mimetype, size }` |
| `manual_entry` | A structured form object for a single transaction entered through the Ledgerise UI |

---

## the four required methods

Every adapter must implement all four methods. Methods not applicable to the adapter's supported modes must still be present and must return a `METHOD_NOT_SUPPORTED` error.

---

### `adapter.meta()`

Returns static metadata about the adapter. Called at registration time. Never called during a live run.

**Returns:**

```json
{
  "name": "paystack-webhook",
  "version": "1.0.0",
  "author": "Your Name or Org",
  "source_system": "paystack",
  "modes": ["webhook"],
  "currency_codes": ["NGN"],
  "docs_url": "https://your-repo/adapters/paystack-webhook/README.md"
}
```

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Unique kebab-case identifier. Must follow the naming convention `{source-system}-{mode}`. Must be unique across all registered adapters. |
| `version` | Yes | Semantic version string. |
| `author` | Yes | Name or GitHub handle. |
| `source_system` | Yes | The upstream system this adapter reads from or writes to. |
| `modes` | Yes | Array of mode strings. At least one required. |
| `currency_codes` | Yes | ISO 4217 codes this adapter can produce. |
| `docs_url` | Yes | Link to the adapter's own README. |

---

### `adapter.validate(input)`

Validates raw input before normalization. Returns a list of validation errors, or an empty list if the input is clean.

This method must be called internally by `normalize()` before any normalization logic runs. It is also called directly by the engine to surface validation errors to operators without triggering normalization.

**Returns on success:**
```json
{ "valid": true, "errors": [] }
```

**Returns on failure:**
```json
{
  "valid": false,
  "errors": [
    {
      "field": "amount",
      "message": "Amount is missing or zero",
      "raw_value": null
    }
  ]
}
```

---

### `adapter.normalize(input)`

The core method. Accepts raw input, validates it, and returns a success or failure envelope.

On success, `records` contains one or more canonical transaction records conforming to `transaction.schema.json v1.0.0`.

**Returns on success:**
```json
{
  "status": "ok",
  "records": [ "...canonical transaction records..." ],
  "cursor": { "..." }
}
```

The `cursor` field is required for poll-mode adapters and must mark progress so the next invocation knows where to start. It is ignored for other modes.

**Returns on failure:**
```json
{
  "status": "error",
  "code": "VALIDATION_FAILED",
  "message": "Human-readable description of what went wrong",
  "raw": "...the original input that caused the failure..."
}
```

The adapter must never throw an unhandled exception. All errors must be caught and returned in the failure envelope.

**Rules inside `normalize()`:**

1. Call `validate(input)` first. If validation fails, return a failure envelope immediately.
2. Generate a new UUID v4 for every record's `id` field.
3. Set `processed_at` to the current UTC timestamp.
4. Set `source.adapter` to the adapter's own `name` from `meta()`.
5. Convert all monetary amounts to the smallest currency unit before emitting (for example, kobo for NGN, cents for USD).
6. Mask sensitive values in `principal.reference`: phone numbers — last 4 digits only; card numbers — last 4 digits only; BVN and NIN — never emit these values.
7. Never emit a record where `source.environment` is `test` without explicitly setting it to `"test"`. Default to `"live"` only when the source data confirms the transaction is real.
8. For failed transactions: normalise and emit them with `status: "failed"` set. Do not filter them out. Add `_adapter_flag: "failed-passthrough"` to the record's `metadata` object.
9. Do not set COA account codes on any record. Account mapping is the engine's responsibility.
10. Do not modify the canonical schema structure. Do not add top-level fields outside the schema. Extra source-specific fields belong in `metadata`.

---

### `adapter.healthcheck()`

Verifies that the adapter can reach its source or destination system. Called at startup and before each engine run.

For **poll adapters**: make a lightweight API call (for example, fetch account info or a single record) and confirm a successful response.

For **webhook adapters**: there is no source to call — return `ok` immediately.

**Returns on success:**
```json
{
  "status": "ok",
  "latency_ms": 142,
  "checked_at": "2026-05-30T08:00:00Z"
}
```

**Returns on failure:**
```json
{
  "status": "error",
  "code": "SOURCE_UNREACHABLE",
  "message": "API returned 503",
  "checked_at": "2026-05-30T08:00:00Z"
}
```

An adapter that fails healthcheck is marked inactive for the current run. The engine logs the failure and retries on the next cycle. It does not crash.

---

## configuration

Adapters must not hardcode credentials, API keys, base URLs, or any operator-specific values. All configuration must be injected at runtime through a config object passed to the adapter at initialization.

Config keys must be in `SCREAMING_SNAKE_CASE`. Sensitive keys must never be logged by the adapter under any circumstances.

Each adapter must document its required and optional config keys in its README. Example:

```json
{
  "PAYSTACK_SECRET_KEY": "sk_live_...",
  "PAYSTACK_WEBHOOK_SECRET": "whsec_...",
  "POLL_INTERVAL_SECONDS": 300,
  "PRODUCT_LINE": "consumer-app"
}
```

---

## naming convention

Adapter names follow the pattern `{source-system}-{mode}`. If an adapter supports multiple modes, create a separate named adapter per mode rather than one adapter handling all modes.

Examples: `paystack-webhook`, `flutterwave-webhook`, `mpesa-poll`, `vtpass-csv`, `interswitch-webhook`.

---

## error codes

| Code | When to use |
|---|---|
| `VALIDATION_FAILED` | Input failed schema validation. Include the `errors` array. |
| `SOURCE_UNREACHABLE` | Could not connect to the source system. |
| `AUTH_FAILED` | Credentials rejected by the source system. |
| `RATE_LIMITED` | Source system returned a rate limit response. |
| `MALFORMED_PAYLOAD` | Input could not be parsed at all (for example, invalid JSON). |
| `UNSUPPORTED_EVENT` | The event type from the source system is not mapped in this adapter. |
| `METHOD_NOT_SUPPORTED` | The called method is not supported by this adapter's modes. |
| `ADAPTER_*` | Custom adapter-specific error. Must be documented in the adapter README. |

---

## deduplication

Adapters are stateless with respect to deduplication. They forward every record they normalise, including potential duplicates. The engine handles deduplication using the `source_id` field.

Adapters must populate `source_id` with the most stable unique identifier available from the source system. If the source provides no stable ID, document this limitation in the README and set `source_id` to null.

---

## what adapters must never do

- Contain journal mapping logic or COA account references
- Write directly to an accounting system
- Modify a canonical record after it has been emitted
- Log raw credentials or secrets
- Swallow errors silently — all errors must surface in the failure envelope
- Emit records that do not conform to `transaction.schema.json v1.0.0`
- Add top-level fields to a canonical record outside the schema
- Assume a default currency without explicit source data confirming it

---

## testing requirements

Every adapter submitted to the Ledgerise registry must include:

1. **Unit tests** for `validate()` and `normalize()` covering at minimum:
   - One valid completed transaction
   - One failed transaction (confirms `status: "failed"` and `_adapter_flag` in metadata)
   - One transaction with missing required fields (confirms the validation failure envelope)
   - One transaction from a test environment (confirms `source.environment: "test"`)

2. **Fixture files** — real or realistic anonymised payloads from the source system, stored in `/fixtures` inside the adapter folder.

3. **A README** documenting supported modes, required and optional config keys, known limitations, the `source_id` strategy used, and any custom type values the adapter emits.

---

## registration

1. Place the adapter in the `/adapters` directory, following the naming convention.
2. The engine calls `adapter.meta()` at startup to register it.
3. The engine calls `adapter.healthcheck()` before the first run.
4. Operators configure which adapters are active and supply config values through the Ledgerise settings UI or config file.

→ See [building an adapter](building-an-adapter.md) for a step-by-step walkthrough
