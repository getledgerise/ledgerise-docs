# sandbox to production

This page is a step-by-step checklist for switching your Ledgerise deployment from sandbox mode to production. Work through it in order. Do not activate your license until every item above it is complete.

---

## why this matters

Going live on a system that is not properly configured — missing mapping rules, misconfigured adapters, unconfigured suspense account — means real transactions will land in the wrong place from day one. Unwinding mispostings from a live accounting system is far more effort than getting the setup right before activation.

Take the time to work through this checklist in sandbox mode. It is designed to be done once.

---

## pre-activation checklist

### environment and infrastructure

- [ ] `DATABASE_URL` points to your production PostgreSQL database, not a local or demo instance.
- [ ] `AUTH_TOKEN_SECRET` is a strong, unique random value — not a placeholder.
- [ ] `LEDGERISE_CREDENTIALS_KEY` is a 64-character hex value — not a placeholder.
- [ ] `VITE_API_BASE_URL` matches the actual public URL of your API.
- [ ] The API is served behind TLS (HTTPS). Do not go live on unencrypted HTTP.
- [ ] The health check endpoint (`/healthcheck`) returns `"repository":"postgres"` and `"db":"ok"`.

### user accounts

- [ ] You have invited at least one named Admin user (not just the default `admin@ledgerise.dev` account).
- [ ] That named Admin can sign in successfully.
- [ ] Finance Officers and Auditors have been invited and can access the dashboard.

> The default `admin@ledgerise.dev` account exists for sandbox setup. It should not be the only Admin in a production deployment.

### adapter configuration

- [ ] At least one inbound adapter (webhook, CSV, or poll) is configured and shows a green healthcheck status in Settings → Adapters.
- [ ] Your outbound adapter (Zoho Books or journal-csv) is configured and shows a green healthcheck status.
- [ ] All adapter credentials are the **production** credentials, not test API keys.

### chart of accounts

- [ ] COA has been imported from your accounting system (Settings → COA Reference → Import COA).
- [ ] The imported account list includes the accounts your finance team expects to use in mapping rules.

### mapping rules

- [ ] A mapping rule exists for every product line and biller combination your platform handles.
- [ ] You have run a test engine cycle in sandbox mode and confirmed entries are posting as expected.
- [ ] The unmapped transaction rate from your test run is zero or explained.

### suspense account

- [ ] A suspense account code is configured in Settings → System. This is the COA account where unmapped transactions will be posted. Without it, unmapped transactions cannot be safely held.

### reconciliation

- [ ] Report sources have been created for each counterparty statement type (Settings → Report Sources).
- [ ] Reconciliation rules have been configured for each report source.

> The posting gate defaults to `disabled`, so reconciliation never blocks journal posting unless you configure it to. That does not make reconciliation optional — it is a standard part of the Ledgerise core loop. Set up at least one report source and one Reference Matching rule before your first real counterparty statement arrives, so you are not reconciling retroactively.

---

## resetting sandbox data

Before activating your license, clear the demo data you accumulated during setup. This ensures your production deployment starts with a clean slate.

Go to **Settings → System → Reset sandbox data** and confirm the reset.

What is cleared:
- All transactions imported during sandbox mode
- All journal entries
- All reconciliation runs, match records, break records, and reports
- All posting batch records
- Audit log entries related to the above

What is **not** cleared:
- Adapter configuration
- Mapping rules
- User accounts
- System settings (COA import, suspense account, etc.)
- Report sources and reconciliation rules

> Once you reset sandbox data and activate your license, there is no path back to sandbox mode without a fresh deployment. Make sure your setup is correct before you proceed.

---

## activation

1. Go to **Settings → System**.
2. Locate the **License** section.
3. Enter your **commercial license key** and **public key** (provided by Ledgerise during onboarding).
4. Click **Activate License**.

[SCREENSHOT: Settings → System showing the license key and public key input fields before activation]

After activation:
- The Sandbox badge disappears from the top navigation bar.
- Settings → System shows **Production License** with your license tier and usage limits.
- The `/healthcheck` endpoint returns `"environment_mode": "production"`.

[SCREENSHOT: Settings → System after activation showing Production License status with no Sandbox badge in the navigation bar]

---

## post-activation verification

Before starting real data imports, verify the activation:

```bash
curl http://localhost:3000/healthcheck
```

Look for `"environment_mode": "production"` in the response. If you still see `"environment_mode": "sandbox"`, the license was not applied correctly. Check that the license key and public key match exactly what Ledgerise provided.

---

## post-activation: first 48 hours

Once real data is flowing, monitor closely:

- **Transactions page** — confirm incoming transaction volume looks correct.
- **Journal Log** — confirm entries are posting on schedule with no failed or retry-exhausted entries.
- **Mapping Rules → Unmapped Today** — any non-zero count means transactions have no matching rule. Create the missing rules immediately.
- **Exceptions badge** (top navigation bar) — aggregates all open issues. A non-zero count needs your attention.

→ See also: [what to monitor after go-live](../01-getting-started/04-quickstart.md#what-to-monitor-after-go-live)
