# access control

This page covers how access to Ledgerise is managed: authentication, session behavior, role-based permissions, and API key security for webhook adapters.

---

## authentication

All access to the Ledgerise dashboard requires a username and password. There is no anonymous access, no public-facing dashboard, and no read-only guest mode. Every session must be authenticated.

Ledgerise uses session-based authentication. After a successful sign-in, a session token is issued and stored as an HTTP-only cookie. The token is not accessible to JavaScript running in the page.

---

## session management

Sessions expire after **8 hours of inactivity**. After that, the user is redirected to the sign-in page on their next action. There is no persistent "remember me" option — each working day requires a new sign-in.

Concurrent sessions from the same account are permitted. Deactivating an account invalidates all active sessions for that user immediately.

---

## role-based access control

Ledgerise has three roles. Each is designed for a specific function in the operator's finance or operations team.

| Role | Transactions | Reconciliation | Mapping Rules | Journal Log | Settings |
|---|---|---|---|---|---|
| **Admin** | Full | Full | Full | Full | Full |
| **Finance** | Full | Full | Full | Full | Read-only |
| **Auditor** | Read-only | Read-only | No access | Read-only | Audit Log only |

**Admin** — full access to all pages and all actions, including user management and system configuration. Every production deployment should have at least two named Admin accounts to avoid a single point of failure for administrative access.

**Finance** — the role for Finance Officers and accountants. Full operational access: mapping rules, reconciliation, journal log, transactions. Cannot change system settings, manage users, or access the audit log.

**Auditor** — read-only access to Transactions, Reconciliation, and Journal Log. Their only Settings access is the Audit Log tab. The Mapping Rules page is not visible to Auditors at all.

→ Full role management guide: [users and roles](../07-settings/02-users-and-roles.md)

---

## user account best practices

**One account per person.** Never share login credentials between team members. The audit log attributes every action to the user who performed it — shared credentials make the audit trail unreliable and remove individual accountability.

**Named Admin accounts before go-live.** The default `admin@ledgerise.dev` account is for sandbox setup only. Before going live, invite at least one named Admin and confirm they can sign in. Do not go live relying on the default account.

**Deactivate promptly.** When a team member leaves or changes role, deactivate or update their account on their last day. Deactivation invalidates their session immediately.

**Minimum two Admins.** Having only one Admin creates a risk: if that person is unavailable, no one can invite new users, change system settings, or access the audit log. Keep at least two named Admin accounts active at all times.

---

## webhook signing secrets

Each inbound webhook adapter has its own signing secret, used to verify that incoming payloads originate from your authorized source system and have not been tampered with.

Webhook signing secrets are:
- Stored encrypted in the Ledgerise database using your `LEDGERISE_CREDENTIALS_KEY`.
- Never logged or displayed in plaintext after initial configuration.
- Independent per adapter — rotating one adapter's secret does not affect others.

**Rotating a signing secret:**

1. Generate a new secret in your source system (for example, in your payment provider's developer console).
2. Go to **Settings → Adapters → your adapter → Configure** and enter the new secret.
3. Save. Ledgerise immediately begins verifying payloads against the new secret.
4. Update the secret in your source system to start sending the new signature.

Configure the new secret in Ledgerise *before* updating your source system. This order avoids a gap where Ledgerise is rejecting valid payloads signed with the new key before Ledgerise knows about it.

Rotation does not require downtime or a restart.

→ See [webhook security](03-webhook-security.md) for payload verification details
