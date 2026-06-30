# license renewal and reissuance

This page covers what to do when a license expires, when you need a new key, or when a key needs to be replaced.

---

## when a license expires

Ledgerise checks license state once per day. On the daily refresh after your license term ends:

- The deployment moves to **read-only mode**.
- All data remains fully accessible — the dashboard, transactions, reconciliation history, journal entries, and exports are all available.
- New transaction ingestion is suspended.
- Journal entry posting is suspended.
- A **License Required** notice replaces the Production badge in the top navigation bar.

Nothing is deleted. Your data is safe. Normal operation resumes as soon as a valid renewed license key is entered.

---

## renewing a license

Contact Ledgerise before your license term ends. Renewal generates a new license key delivered through the same [one-time retrieval flow](03-activating-your-license.md#how-your-license-key-is-delivered) used for the initial activation.

Once you have the new key:

1. Go to **Settings → System → License**.
2. Enter the new license key and public key.
3. Click **Activate License**.

The renewed license takes effect at the next daily refresh. If your deployment was already in read-only mode due to expiry, ingestion and posting resume after the refresh.

---

## if you lose your key before copying it

The one-time retrieval page cannot be accessed again after it expires or after the keys are copied. If you closed the page without saving the keys:

Contact Ledgerise. The existing keys are revoked and a new set is issued via a fresh retrieval link. Your deployment continues operating on the existing license until the new keys are entered — revoking the old key does not immediately interrupt service. The revocation takes effect at the next daily license refresh.

---

## if you suspect a key has been compromised

Contact Ledgerise immediately. Provide the approximate time you suspect the key was exposed. Ledgerise will:

1. Revoke the compromised key. The revocation takes effect at the next daily license refresh.
2. Issue a replacement key via a new one-time retrieval link.

Enter the replacement key in **Settings → System → License** as soon as you receive it. If there is a gap between revocation and the new key being entered, your deployment may briefly enter read-only mode on the daily refresh. This is the expected behavior — it ensures a compromised key cannot be used to activate another deployment.

---

## reissuance for a new deployment

If you are setting up a second Ledgerise deployment — for example, a disaster recovery environment or a regional instance — contact Ledgerise. Each production deployment requires its own license key. A key issued for one deployment cannot be used to activate another.

---

## summary

| Situation | What to do |
|---|---|
| License expiring soon | Contact Ledgerise to renew before the expiry date |
| License already expired (read-only mode) | Contact Ledgerise, enter the new key in Settings → System |
| Key lost before copying | Contact Ledgerise — old key is revoked, new key issued |
| Key suspected compromised | Contact Ledgerise immediately for emergency revocation and replacement |
| New deployment needs a key | Contact Ledgerise — each deployment requires its own key |
