# building an adapter

This walkthrough takes you from nothing to a registered, tested inbound adapter. It assumes you have read the [adapter specification](02-adapter-spec.md) and understand the four required methods.

---

## when to build a custom adapter

Use the built-in **generic-webhook**, **csv-import**, or **poll-adapter** whenever your source system outputs standard JSON or CSV with a straightforward field structure. They cover most cases and require no code.

Build a custom adapter when:

- The source system has a proprietary wire format, non-JSON encoding, or unusual authentication scheme that the generic adapters cannot handle through field mapping alone.
- The source system requires request signing or a custom auth handshake that the generic adapters don't support.
- You have specific deduplication logic — for example, a source system that re-sends records with the same ID but different states and you need custom handling to pick the right one.
- You need custom type mapping logic: the source system's event taxonomy doesn't map cleanly to the standard transaction type taxonomy and requires conditional logic.

---

## step 1 — choose the adapter mode

Decide whether your adapter will be webhook, poll, or file import (or a combination). This determines the shape of your `normalize()` input.

| Mode | Choose when |
|---|---|
| Webhook | The source system pushes events to a URL on each transaction |
| Poll | The source system exposes an API you call to fetch recent transactions |
| File import | Operators upload batch exports from the source system |

Create one adapter per mode. Avoid a single adapter that handles multiple modes — it becomes harder to test and maintain.

---

## step 2 — implement `adapter.meta()`

This is the simplest method. Return static metadata describing your adapter.

```javascript
adapter.meta = () => ({
  name: "acme-webhook",           // kebab-case: {source-system}-{mode}
  version: "1.0.0",
  author: "your-name",
  source_system: "acme-payments",
  modes: ["webhook"],
  currency_codes: ["NGN"],
  docs_url: "https://your-repo/adapters/acme-webhook/README.md"
});
```

The `name` must be unique across all registered adapters and follow the `{source-system}-{mode}` pattern.

---

## step 3 — implement `adapter.validate(input)`

Validate the raw input before any normalization happens. Check that required fields are present and in the expected format. Return a clean errors array if everything is valid; return a list of error objects if not.

```javascript
adapter.validate = (input) => {
  const errors = [];

  if (!input.transaction_id) {
    errors.push({ field: "transaction_id", message: "Missing transaction ID", raw_value: input.transaction_id });
  }
  if (!input.amount || input.amount <= 0) {
    errors.push({ field: "amount", message: "Amount is missing or zero", raw_value: input.amount });
  }
  // ... more checks

  return { valid: errors.length === 0, errors };
};
```

Be specific in your error messages — operators will read them in the Transactions page when a payload fails.

---

## step 4 — implement `adapter.normalize(input)`

The core method. Call `validate()` first, then map source fields to the canonical schema.

```javascript
adapter.normalize = (input) => {
  const validation = adapter.validate(input);
  if (!validation.valid) {
    return { status: "error", code: "VALIDATION_FAILED", message: "Validation failed", raw: input };
  }

  const record = {
    id: uuidv4(),                              // Rule 2: always generate a new UUID
    source_id: input.transaction_id,           // most stable unique ID from the source
    processed_at: new Date().toISOString(),    // Rule 3: current UTC timestamp
    occurred_at: input.created_at,
    completed_at: input.settled_at ?? null,
    status: mapStatus(input.status),           // map source status to canonical values
    amount: Math.round(input.amount * 100),    // Rule 5: convert to smallest unit (kobo)
    currency: "NGN",
    direction: "inbound",
    type: mapType(input.event_type),
    product: {
      line: input.product_line,
      biller: input.biller ?? null,
      biller_category: input.biller_category ?? null
    },
    principal: {
      reference: maskPhone(input.customer_phone) // Rule 6: mask PII
    },
    source: {
      adapter: "acme-webhook",                  // Rule 4: adapter's own name
      system: "acme-payments",
      environment: input.is_test ? "test" : "live"  // Rule 7: never default to live without confirmation
    },
    metadata: {}
  };

  // Rule 8: pass failed transactions through, don't drop them
  if (record.status === "failed") {
    record.metadata._adapter_flag = "failed-passthrough";
  }

  return { status: "ok", records: [record] };
};
```

**Key rules in `normalize()`:**
- Rules 9 and 10: do not set COA account codes, do not add top-level fields outside the schema.
- Rules 2–8: see the [adapter specification](02-adapter-spec.md#normalizeinput) for the full list.

---

## step 5 — implement `adapter.healthcheck()`

For a webhook adapter, return `ok` immediately — there is no source to call:

```javascript
adapter.healthcheck = () => ({
  status: "ok",
  latency_ms: 0,
  checked_at: new Date().toISOString()
});
```

For a poll adapter, make a lightweight API call:

```javascript
adapter.healthcheck = async () => {
  const start = Date.now();
  try {
    await acmeClient.getAccountInfo();
    return { status: "ok", latency_ms: Date.now() - start, checked_at: new Date().toISOString() };
  } catch (err) {
    return { status: "error", code: "SOURCE_UNREACHABLE", message: err.message, checked_at: new Date().toISOString() };
  }
};
```

---

## step 6 — write tests

Your adapter must include unit tests covering at minimum:

1. A valid completed transaction — `normalize()` returns `status: "ok"` with a correctly-formed canonical record.
2. A failed transaction — record has `status: "failed"` and `metadata._adapter_flag === "failed-passthrough"`.
3. Missing required fields — `validate()` returns `valid: false` with a non-empty `errors` array.
4. A test-environment transaction — record has `source.environment: "test"`.

---

## step 7 — add fixture files

Create a `/fixtures` directory inside your adapter folder. Add real or realistic anonymised payloads from the source system — one file per test scenario. Use these fixtures in your tests so they are grounded in actual source data, not invented shapes.

---

## step 8 — write a README

Your adapter's README must document:

- Supported modes
- Required and optional config keys (in `SCREAMING_SNAKE_CASE`)
- Any known limitations or edge cases
- The `source_id` strategy you used, and what to do if the source system provides no stable ID
- Any custom `type` values the adapter emits that fall outside the standard taxonomy, and why

---

## step 9 — register the adapter

1. Place the adapter folder in the `/adapters` directory, following the `{source-system}-{mode}` naming convention.
2. On the next Ledgerise startup, the engine calls `adapter.meta()` and registers it.
3. The engine calls `adapter.healthcheck()` before the first run.
4. Go to **Settings → Adapters** in the Ledgerise UI, find your adapter, enter its config values, and enable it.

→ See [adapter specification § registration](02-adapter-spec.md#registration) for full details
