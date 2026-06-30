# system settings

Settings → System is where Admins configure the engine, manage the license, and reset data. Finance Officers have read-only access to most of this tab.

---

## engine schedule

The journal engine runs on a cron schedule. The default is every hour (`0 * * * *`), which means the engine starts a new run at the top of each hour.

Change this in **Settings → System → Engine Schedule**. Use standard 5-field cron syntax. A shorter interval means transactions are posted more frequently; a longer interval reduces the number of API calls made to your accounting system per day.

> If you reduce the interval significantly (for example, to every 5 minutes), make sure your accounting system's API rate limits can sustain the increased call volume.

---

## batch size

The maximum number of transactions the engine processes in a single run. The default is 500. If a run picks up more than the batch size, the remainder are held and picked up on the next scheduled run.

Increase the batch size if you have high-volume operations and your infrastructure can handle larger runs. Decrease it if individual engine runs are timing out or hitting API rate limits.

---

## suspense account code

The COA account code where the engine posts transactions that cannot be matched to a mapping rule. Every unmapped transaction goes here rather than being dropped.

Set this before going live — the engine cannot safely handle unmapped transactions without it. The code must match an account that exists in your accounting system, so import your COA first and then copy the code from the account list.

→ See [chart of accounts](../05-mapping-rules/chart-of-accounts.md)

---

## retry policy

The maximum number of retry attempts and the backoff strategy for failed journal entries. The default is 5 attempts with exponential backoff (5 min → 15 min → 1 hr → 4 hr → 24 hr). After the final attempt, entries are marked `retry_exhausted` and require manual intervention.

Adjust the max attempts or backoff timing here if your accounting system is frequently unavailable for extended periods.

---

## posting gate

Controls whether the journal engine waits for a transaction to clear reconciliation before generating its journal entry.

| Value | Behaviour |
|---|---|
| `disabled` (default) | The engine posts from all completed, unposted transactions regardless of reconciliation status |
| `provider_matched_or_better` | Holds entries until the transaction is at least matched against a provider statement |
| `bank_matched_or_better` | Holds entries until the transaction is matched against a bank statement |
| `full_match_or_resolved` | Holds entries until the transaction is fully matched on all configured sources, or a break is manually resolved |

Leave this at `disabled` if you want reconciliation to run independently of posting. Enable it only if your compliance or control requirements mean you must not post until a transaction is verified against an external counterparty.

---

## AI provider

Natural language features in Ledgerise — including the natural language report mode in Reconciliation — require an AI provider API key. Enter your key here. The key is stored encrypted and is never logged.

If no key is configured, natural language features are hidden from the UI.

---

## report sources

**Settings → Report Sources** lists the saved counterparty statement identities used in reconciliation. A report source combines a counterparty name and a statement type into a display label — for example, `Paystack — Settlement Report` or `GTBank — Collection Account Statement`.

Report sources are created automatically the first time you import a statement of that type. You can rename them or delete unused ones here. Reconciliation rules are organised by report source, so deleting a report source also removes its associated rules.

→ See [reconciliation overview](../04-reconciliation/overview.md) for how report sources are used

---

## resetting sandbox data

**Settings → System → Reset sandbox data** clears all transactional data accumulated during sandbox setup: transactions, journal entries, reconciliation runs, match and break records, and audit log entries related to those operations. Adapter configuration, mapping rules, user accounts, system settings, and report sources are preserved.

Do this once, just before activating your commercial license, to start production with a clean slate.

> This action is permanent. There is no undo.

---

## license and production activation

Enter your **commercial license key** and **public key** (provided by Ledgerise during onboarding) in the License section at the bottom of Settings → System. Click **Activate License**.

After activation:
- The Sandbox badge disappears from the top navigation bar.
- The deployment is in production mode. Journal entries will now be submitted to your accounting system.
- The `/healthcheck` endpoint returns `"environment_mode": "production"`.

Once activated, there is no path back to sandbox mode without a fresh deployment.

→ Full checklist: [sandbox to production](../02-deployment/sandbox-to-production.md)

[SCREENSHOT: Settings → System showing the engine schedule, batch size, and suspense account code fields]

[SCREENSHOT: Settings → System showing the License section with the license key and public key input fields before activation]
