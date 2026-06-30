# users and roles

Settings → Users is where Admins manage who has access to Ledgerise and what they can do.

---

## who can access this page

Only **Admins** can invite users, change roles, and deactivate accounts. Finance Officers and Auditors do not have access to the Users tab.

---

## the user list

The page shows all user accounts in your deployment with columns for name, email, role, last login, and status (active or deactivated).

[SCREENSHOT: Settings → Users showing the user list with name, email, role badges, last login timestamps, and the Invite User button in the page header]

---

## the three roles

| Role | Transactions | Reconciliation | Mapping Rules | Journal Log | Settings |
|---|---|---|---|---|---|
| **Admin** | Full | Full | Full | Full | Full |
| **Finance** | Full | Full | Full | Full | Read-only (Adapters, COA Reference, System only — not Users or Audit Log) |
| **Auditor** | Read-only | Read-only | No access | Read-only | Audit Log only |

**Admin** — full access to every page and every action, including system configuration, license management, and user management. At least one named Admin is required before going live.

**Finance** — the role for Finance Officers and accountants. Full operational access: they can create and edit mapping rules, trigger engine runs, retry failed entries, run reconciliation, and resolve breaks. Within Settings, they have read-only access to Adapters, COA Reference, and System. They cannot access the Users or Audit Log tabs, and cannot change any settings.

**Auditor** — read-only access to Transactions, Reconciliation, and Journal Log. Their only Settings access is the Audit Log tab. They cannot create, edit, or delete anything. Ledgerise does not show the Mapping Rules page to Auditor accounts at all.

---

## inviting a user

1. Go to **Settings → Users** and click **Invite User**.
2. Enter the user's email address and select their role.
3. Click **Send Invite**. Ledgerise sends the user an email with a sign-in link and their temporary credentials.

The invited user appears in the list immediately with status `pending` until they sign in for the first time.

---

## changing a role

1. Click the **Edit** icon on the user's row.
2. Select a new role from the role picker modal.
3. Confirm. The change takes effect immediately on the user's next page load — their active session is not invalidated.

[SCREENSHOT: Role picker modal showing the three role options (Admin, Finance, Auditor) with short descriptions of each]

---

## deactivating a user

Click **Deactivate** on the user's row. The user's session is invalidated immediately — they are signed out and cannot sign in again. Their account record and all historical actions attributed to them remain in the system and the audit log.

Deactivated accounts can be reactivated by an Admin if needed.

> Do not deactivate the only Admin account. Ledgerise prevents this, but always confirm that at least one other Admin can sign in before deactivating an account.

---

## best practices

**One account per person.** Every team member should have their own named account. Do not share login credentials. The audit trail records every action against the user who performed it — sharing logins makes the audit trail unreliable and removes individual accountability.

**Deactivate promptly.** When a team member leaves, deactivate their account on their last day. This is especially important for Admin accounts.

**Invite Admins first.** Before going live, make sure at least one named Admin (not the default `admin@ledgerise.dev` account) can sign in and take ownership of the deployment.

→ See [sandbox to production](../02-deployment/05-sandbox-to-production.md) for the full pre-activation checklist
