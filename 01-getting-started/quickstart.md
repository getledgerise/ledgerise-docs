# quickstart

This guide takes you from a fresh Ledgerise deployment to your first posted journal entry. Steps are numbered and sequential. Each step links to the full guide where you need more detail.

**Who this is for:** Admins completing the initial setup. If you are a Finance Officer joining an already-configured Ledgerise deployment, you can start directly with [mapping rules](../05-mapping-rules/overview.md) or [reconciliation](../04-reconciliation/overview.md).

**Expected time:** Most operators complete initial setup in 2–4 hours. The bulk of that time is spent configuring mapping rules with your finance team.

---

## before you begin

Make sure you have the following ready:

- A running Ledgerise deployment. If you have not deployed yet, start with the [deployment overview](../02-deployment/overview.md).
- Your Ledgerise commercial license key and public key, provided by Ledgerise via the key delivery flow.
- Credentials for your accounting system. For Zoho Books: your Client ID, Client Secret, and Organization ID.
- Access to your accounting system to view your chart of accounts.
- A list of your transaction product lines and billers — you will need these when creating mapping rules.
- A sample statement (CSV) from your primary payment provider or bank — you will use this to set up reconciliation.

---

## step 1 — sign in to the sandbox

Open your Ledgerise dashboard URL in a browser. A fresh deployment starts in **sandbox mode**, which means it is safe to experiment — nothing you do here will affect your accounting system or your production data.

Sign in with the default sandbox credentials:

- **Email:** `admin@ledgerise.dev`
- **Password:** `password`

You should see the Ledgerise dashboard with a **Sandbox** badge in the top navigation bar. That badge confirms you are in a safe, isolated mode.

> Before going live, you will invite your team members and change the default admin credentials. The default account is for sandbox setup only.

[SCREENSHOT: Ledgerise dashboard after first login showing the Sandbox badge in the top navigation bar and the Transactions page as the default landing view]

---

## step 2 — connect your accounting system

Before Ledgerise can post journal entries anywhere, it needs to know where to send them.

1. Go to **Settings → Adapters**.
2. Find the outbound adapter for your accounting system. If you use Zoho Books, click **Configure** on the `zoho-books` adapter.
3. Enter the required credentials and save.
4. The adapter runs a healthcheck. A green status indicator confirms the connection is working.

If you are not yet ready to connect your accounting system, use the `journal-csv` adapter instead. It exports generated entries as a CSV file you can import manually — a useful fallback for any accounting system.

→ Full instructions: [zoho-books adapter](../08-adapters/zoho-books.md) | [journal csv export](../08-adapters/journal-csv-export.md)

[SCREENSHOT: Settings > Adapters showing the zoho-books adapter tile with a green healthcheck status badge after successful configuration]

---

## step 3 — import your chart of accounts

Ledgerise needs a copy of your chart of accounts so your finance team can select the right accounts when creating mapping rules.

1. Go to **Settings → COA Reference**.
2. Click **Import COA**.
3. Ledgerise pulls your account list from the connected accounting system. Your accounts appear within a few seconds, listed with their codes, names, and types.

If you are using the `journal-csv` adapter and there is no direct accounting system connection, skip this step for now. You will enter account codes manually when creating mapping rules.

→ Full instructions: [chart of accounts](../05-mapping-rules/chart-of-accounts.md)

---

## step 4 — configure an inbound adapter

An inbound adapter is how transaction data enters Ledgerise. Choose the one that matches how your transaction system sends data:

| Your source system can... | Adapter to use |
|---|---|
| Push transaction events to a URL in real time | Webhook adapter |
| Export transactions as a CSV or XLSX file | CSV import adapter |
| Expose a JSON API you can call on a schedule | Poll adapter |

To configure an inbound adapter:

1. Go to **Settings → Adapters**.
2. Click **Configure** on your chosen adapter.
3. Fill in the endpoint, credentials, and field mapping as prompted.
4. Enable the adapter using the toggle. A green healthcheck status confirms it is active.

→ Full instructions: [webhook adapter](../08-adapters/generic-webhook.md) | [csv import](../08-adapters/generic-csv.md) | [poll adapter](../08-adapters/generic-poll.md)

Once configured, transactions from your source system will begin appearing in the **Transactions** page.

---

## step 5 — create your first mapping rules

This is the most important setup step. Mapping rules tell Ledgerise which accounting accounts to debit and credit for each type of transaction. Without them, all transactions land in the suspense account.

Work through this step with your finance officer or accountant. For each combination of product line and biller category that your platform handles, you will create one rule.

To create a rule:

1. Go to **Mapping Rules** and click **Add Rule**.
2. Select the **product line** — for example, `bill-payment`.
3. Optionally add a **biller category** (for example, `electricity`) or a specific **biller** (for example, `ikeja-electric`) for more granular control.
4. Select the **debit account** using the COA picker.
5. Add one or more **credit accounts** with percentage splits that sum to 100.
6. Set the rule status to **Active** and save.

Repeat for each product line and biller your platform handles.

> **Start broad, then go specific.** Create catch-all rules for each product line first, then add more specific rules for billers that need different accounting treatment. The engine always uses the most specific rule that matches.

→ Full instructions: [creating a rule](../05-mapping-rules/creating-a-rule.md) | [rule resolution order](../05-mapping-rules/rule-resolution-order.md)

[SCREENSHOT: Mapping Rules page showing several completed rules with colour-coded account code chips (blue for asset, green for income), and the Add Rule button at top right]

---

## step 6 — set up reconciliation

Reconciliation verifies your internal transaction records against external statements from your payment providers and banks. Set up the basics now, before live data starts flowing in.

1. Go to **Reconciliation → Import Statement** and import your sample provider statement.
2. Ledgerise creates a **report source** for it automatically — a saved identity combining the counterparty and statement type, for example `Paystack — Settlement Report`.
3. Go to **Reconciliation → Rules** and configure at least one **Reference Matching** rule for that report source. This is the minimum required for the matching engine to pair internal and external records.
4. Run the import again (or trigger a reconciliation run) and check the resulting match rate before you go live.

By default, the posting gate is **disabled** — reconciliation status does not block journal posting, and the engine continues to post completed transactions on its normal schedule regardless of match status. You can tighten this later in Settings → System if you want posting to wait on reconciliation.

→ Full instructions: [reconciliation overview](../04-reconciliation/overview.md) | [reconciliation rules](../04-reconciliation/reconciliation-rules.md)

---

## step 7 — import a test batch or trigger a webhook

Now that adapters and rules are configured, bring in some transaction data to verify the setup.

**If you are using the CSV adapter:** Go to Settings → Adapters → csv-import → **Upload File**. Upload a sample export from your transaction system. A small batch of 20–50 transactions is enough to verify your setup.

**If you are using the webhook adapter:** Send a test payload from your source system to the webhook URL shown in the adapter configuration panel.

**If you are using the poll adapter:** The first poll will run on your configured schedule. You can also trigger it immediately from Settings → Adapters → poll-adapter → **Run Now**.

After a few moments, go to the **Transactions** page. Your imported records should appear with a posting status of `unposted`.

---

## step 8 — run the engine and check the journal log

The journal engine runs on its configured schedule. For your first test, trigger a manual run:

1. Go to **Journal Log**.
2. Click **Run Engine Now**.
3. The engine processes your unposted transactions, applies your mapping rules, and generates journal entries.
4. Within a minute or two, entries appear in the table.

Here is what each outcome means and what to do:

| Entry status | What it means | What to do |
|---|---|---|
| `posted` (green) | The entry was submitted to your accounting system successfully | Nothing — this is the goal |
| `unmapped` (amber) | No matching mapping rule was found | Click **Assign Rule** → create a rule for that product line and biller, then run the engine again |
| `failed` (red) | The accounting system returned an error | Open the detail drawer → read the error in the posting history → fix the root cause → click **Retry** |

→ Full instructions: [retrying failed entries](../06-journal-log/retrying-failed-entries.md)

[SCREENSHOT: Journal Log after the first engine run showing a mix of green posted and amber unmapped entries, with the Run Engine Now button visible in the page header]

---

## step 9 — invite your team

Once entries are posting correctly, bring in the rest of your team before going live.

1. Go to **Settings → Users** and click **Invite User**.
2. Assign the **Finance** role to your accountants and Finance Officers — they get full access to Transactions, Reconciliation, Mapping Rules, and Journal Log, and read-only access to Settings.
3. Assign the **Auditor** role to anyone who only needs read-only access.
4. Confirm that at least one named Admin can sign in before you reset the sandbox.

→ Full instructions: [users and roles](../07-settings/users-and-roles.md)

---

## step 10 — go live

When you are satisfied that transactions are flowing in and journal entries are posting correctly, follow these steps to switch from sandbox to production:

1. **Configure your suspense account code** in Settings → System. This is the COA account where unmapped transactions will land.
2. **Reset sandbox data** in Settings → System → Reset sandbox data. This clears all demo transactions, journal entries, reconciliation runs, and related records — leaving your adapter configuration, mapping rules, and users intact.
3. **Activate your commercial license** in Settings → System. Enter your license key and public key. The Sandbox badge disappears and the status reads **Production License**.
4. **Verify the activation** by visiting `/healthcheck`. It should return `environment_mode: "production"`.
5. Start your real data imports or live webhook connections.

→ Full checklist: [sandbox to production](../02-deployment/sandbox-to-production.md)

[SCREENSHOT: Settings > System after production activation showing the Production License status indicator and no Sandbox badge in the top navigation bar]

---

## what to monitor after go-live

For the first 48–72 hours, check the following each day:

- **Transactions page stat bar** — is today's transaction count what you expect?
- **Mapping Rules → Unmapped Today** — if this is non-zero, you have transactions without a matching rule. Create the missing rules.
- **Journal Log** — are entries posting on schedule? Are there any failed or retry-exhausted entries that need attention?
- **Reconciliation → Breaks** — are breaks being raised and resolved at a reasonable pace? A growing backlog of open breaks usually means a reconciliation rule needs adjusting.
- **Exceptions badge** in the top navigation bar — this count aggregates all open issues across unmapped transactions, failed postings, and reconciliation breaks. A non-zero count means something needs your attention.

Most operators reach a steady state within the first week, with an unmapped rate below 2% of daily transaction volume.
