# activating your license

License activation switches your Ledgerise deployment from sandbox to production mode. Do this once — after your setup is complete, your team is invited, and your sandbox testing is done.

---

## before you activate

Work through this checklist first. Activation is permanent for the current deployment — there is no path back to sandbox mode without a fresh install.

- [ ] All adapters are configured with **production credentials** (not test API keys).
- [ ] Mapping rules are in place for all product lines and billers your platform handles.
- [ ] Your suspense account code is set in Settings → System.
- [ ] Reconciliation report sources and at least one rule set are configured.
- [ ] At least one named Admin account (not `admin@ledgerise.dev`) can sign in.
- [ ] Finance Officers and Auditors have been invited and can access the dashboard.
- [ ] Sandbox data has been reset: **Settings → System → Reset sandbox data**.

→ Full pre-activation checklist: [sandbox to production](../02-deployment/05-sandbox-to-production.md)

---

## how your license key is delivered

Ledgerise delivers license keys through a secure one-time retrieval flow:

1. Ledgerise sends you an email with a **one-time retrieval link**. The link is valid for a limited time.
2. Your **client ID** is sent to you separately — by phone, WhatsApp, or direct message — never in the same email as the link.
3. Visit the retrieval link and enter your client ID.
4. Copy the **license key** and the **license public key** displayed on the page.
5. The page expires after you copy the keys. It cannot be accessed again.

Keep both keys somewhere safe before closing the retrieval page. If you lose them before entering them, contact Ledgerise — the existing keys will be revoked and new ones reissued.

> Treat the license key like a database password. Enter it only in Settings → System. Do not put it in code, config files, environment variable documentation, Slack messages, or emails.

---

## activation steps

1. Go to **Settings → System**.
2. Scroll to the **License** section.
3. Enter your **commercial license key** in the first field.
4. Enter the **license public key** in the second field.
5. Click **Activate License**.

[SCREENSHOT: Settings → System showing the License section with the license key and public key input fields before activation]

---

## confirming activation

After clicking Activate License, verify two things:

**In the dashboard:**
- The **Sandbox** badge in the top navigation bar disappears.
- Settings → System → License shows **Production License** with your tier name and a valid license state.

**Via the healthcheck endpoint:**

```bash
curl https://your-ledgerise-domain/healthcheck
```

Look for:

```json
{
  "environment_mode": "production",
  "license": {
    "state": "active",
    "tier": "Pro"
  }
}
```

If `environment_mode` still reads `"sandbox"`, the license was not applied. Double-check that you entered both keys exactly as they appeared on the retrieval page — no leading or trailing spaces.

[SCREENSHOT: Settings → System after successful activation showing Production License status and the environment mode indicator]

---

## after activation

Your deployment is now live. Journal entries will be submitted to your accounting system on the next engine run.

- Start your real data imports or switch your payment webhooks from test to production endpoints.
- Monitor the first 48–72 hours closely. → See [what to monitor after go-live](../01-getting-started/04-quickstart.md#what-to-monitor-after-go-live)
