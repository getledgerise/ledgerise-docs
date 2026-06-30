# audit log

The Audit Log is a system-wide, immutable record of every user action and system event in Ledgerise. It is the evidence layer that tells you who did what, and when.

---

## where to find it

Go to **Settings → Audit Log**.

This is one of the few parts of Settings that Auditors can access. Admins have full access; Finance Officers do not have access to the Audit Log tab.

---

## what gets recorded

Every significant action in Ledgerise writes an entry to the Audit Log automatically. There is no configuration required and no way to opt out of logging for individual actions. Recorded events include:

**Mapping rules**
- Rule created
- Rule updated
- Rule activated
- Rule deactivated

**Reconciliation**
- Statement imported (reconciliation run initiated)
- Break resolved
- Exception resolved

**Transactions and journal entries**
- Flag assigned
- Flag resolved
- Manual posting override

**Users and access**
- User invited
- Role changed
- User deactivated

**System**
- System settings changed (engine schedule, batch size, suspense account, retry policy, posting gate)

---

## the log table

Each row in the Audit Log shows:

| Column | What it contains |
|---|---|
| Timestamp | The exact date and time the event occurred (UTC) |
| Event type | A structured label for the action — for example, `mapping_rule.deactivated` or `user.role_changed` |
| Actor | The user who performed the action, or `system` for automated engine actions |
| Target | The ID of the object affected — a rule ID, transaction ID, break ID, user ID, and so on |
| Summary | A human-readable description of what changed |

[SCREENSHOT: Settings → Audit Log showing several event rows with event type, actor name, target ID, and timestamp columns, and the filter bar at the top]

---

## filtering the log

Use the filter bar to narrow by:

- **Event type** — select one or more event types from the taxonomy
- **Actor** — filter to a specific user to see all actions attributed to them
- **Date range** — select a start and end date

For a period-specific audit — for example, reviewing all actions taken during a month-end close — set a date range and export the filtered results.

---

## immutability

The Audit Log is append-only. Entries cannot be edited, deleted, or suppressed through the UI or any API — not even by an Admin. Every row is a permanent record of what happened.

This is a design requirement. Any system that allows its own audit log to be modified cannot be used as evidence.

---

## how auditors use it

Auditors access the Audit Log from Settings → Audit Log — this is the only part of Settings visible to the Auditor role. Common use cases:

**Period review** — filter by date range to see all system activity during a period of interest. A clean log (no unexpected manual overrides, no unexplained rule changes) is the first thing an auditor checks before accepting financial records.

**Specific-action tracing** — filter by event type and actor to answer a specific question: "Who deactivated the electricity revenue mapping rule, and when?" The log has the answer with a timestamp and user attribution.

**Change control verification** — confirm that rule changes happened in the right sequence and were made by authorised users. Cross-reference with the [rule audit trail](../05-mapping-rules/05-rule-audit-trail.md) for the before-and-after state of each rule change.
