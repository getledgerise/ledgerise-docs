# commercial tiers

Commercial licenses are priced by the number of transaction records Ledgerise ingests per calendar month — **monthly imported source rows**.

---

## tier limits

| Tier | Monthly imported source rows |
|---|---|
| Starter | Up to 30,000 |
| Pro | Up to 100,000 |
| Scale | Up to 300,000 |
| Enterprise | Custom volume — contact Ledgerise |

These limits apply to inbound ingestion across all adapters combined. Each unique transaction record counts once when it is first ingested, regardless of how many engine runs process it or how many journal entries it produces.

---

## checking your current usage

**Via the healthcheck endpoint:**

```bash
curl https://your-ledgerise-domain/healthcheck
```

After production activation, the response includes:

```json
{
  "environment_mode": "production",
  "license": {
    "state": "active",
    "tier": "Pro",
    "usage": {
      "current_month": 41823,
      "limit": 100000,
      "percent_used": 41.8
    }
  }
}
```

**Via the dashboard:**

Go to **Settings → System → License**. The license section shows your current tier, this month's ingestion count, your limit, and the last time the license state was refreshed.

---

## what happens when you approach your limit

Ledgerise refreshes license state once per day. There is no warning or throttling as you approach the limit during a month — ingestion continues normally until the daily refresh detects that the limit has been exceeded.

When usage exceeds the tier limit at the next daily refresh, the deployment moves to **read-only mode**: ingestion and posting are suspended until a valid license covering higher volume is activated.

If your volume is predictable, monitor the `percent_used` value from the healthcheck weekly and upgrade before you reach 100%. If volume spikes unexpectedly, contact Ledgerise as soon as you notice — read-only mode can be resolved the same day a new key is issued.

---

## upgrading your tier

Contact Ledgerise to upgrade. A new license key covering the higher tier is issued and delivered via the same secure delivery flow used for initial activation. Enter the new key in **Settings → System** — no other changes are required. The tier upgrade takes effect at the next daily license refresh.

---

## implementation fee

A one-time implementation fee is charged separately from the license for commercial on-premise deployments. It covers:

- Scoping your transaction volume, product lines, and adapter requirements
- Adapter configuration and field mapping for your source systems
- Mapping rule setup with your finance team
- Go-live support and post-activation monitoring

The implementation fee is quoted during pre-sales scoping and is not optional — it reflects the services work required to make an on-premise deployment correct and sustainable from day one. For a mature fintech with several product lines and dozens of billers, mapping rule setup alone typically involves 50–100 rules.

Contact Ledgerise during the sales process to discuss the implementation scope and fee for your specific deployment.
