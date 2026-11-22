# Rolling out FIDO2 security keys

A short walkthrough of how I would roll out FIDO2 security keys in Microsoft Entra ID as a phishing-resistant sign-in method for the users and roles where they fit best.

---

## 1. Context

Most users are fine with a phone-based method, but some groups are better served by a hardware security key:

- **Privileged and regulated roles** that need a stronger, controlled credential than a phone prompt.
- **Users without a suitable phone,** or in areas where phones are not allowed.
- **Shared workstations, kiosks and frontline roles,** where the credential should belong to the person, not a phone or a shared account.
- **Users who will not put a work app on a personal phone.**

A FIDO2 key is a device-bound, phishing-resistant passkey. It works without a phone, and IT can issue, restrict and recover it.

There is also a deadline that makes this more urgent. Microsoft is retiring Microsoft-provided SMS and voice authentication from February 2027, and its recommended replacement is passkeys. Users who rely on SMS or voice and cannot use the Authenticator app are the clearest case for a key. But the rollout is worth doing on its own merits for the groups above, and it also moves the organisation towards phishing-resistant authentication generally.

## 2. Goal

Put a phishing-resistant, device-bound sign-in method in the hands of the users who need it most, without locking anyone out and without creating a lost-key support burden.

Specifically, it set out to:

- Move privileged and regulated roles onto approved, device-bound keys.
- Give users who cannot use a phone-based method a working alternative, including those affected by the SMS and voice retirement.
- Keep recovery simple, so a lost key is a short call to the service desk and not a lockout.

## 3. Approach

**Identify who needs a key.** Build a named list of users with the reason each one needs a key: privileged role, regulated role, no suitable phone, shared workstation, or an objection to the app. The reason sets the priority and the handling. Users who relied on SMS or voice and cannot use the Authenticator app go in the same list.

**Configure the policy.**

- Enable Passkey (FIDO2) in the Authentication methods policy with self-service setup, scoped to a group.
- Use a separate passkey profile for key users. If attestation is enabled in a profile, only device-bound passkeys are allowed, so synced passkeys are excluded. That suits privileged and regulated roles, where I would also restrict keys to approved models.
- Check Conditional Access and any custom authentication strengths allow FIDO2. A strength that allows other methods but not FIDO2 keeps users blocked even after they register a key.

**Choose and stock the keys.** Pick one or two approved models, order enough for the users plus spares, and allow for lead times. Issue a **second key** or a second method to critical and privileged users.

**Register the keys.** Users register at their desk, or with the service desk for those who cannot. Where a user cannot complete the normal proof-up step, a Temporary Access Pass can be used to register the key (see [TAP Rollout](https://github.com/ejneedham00/Entra-Authentication-Methods/blob/main/Temporary%20Access%20Pass%20Rollout.md)).

**Phase it.**

| Phase | What happens | Gate to move on |
| --- | --- | --- |
| 1. Identify | Named list of users and reasons | List agreed |
| 2. Prepare | Passkey profile, Conditional Access check, key model chosen, stock ordered, Standard Operating Procedure (SOP) written | Test user can register a key and sign in |
| 3. Pilot | A few users across the scenarios, including a privileged user and a shared-workstation user | Standard Operating Procedure (SOP) and lost-key process proven |
| 4. Waves | Issue keys by group, privileged and regulated roles first, then users affected by the SMS and voice retirement, then the rest | Registration and tickets reviewed per wave |
| 5. Clean-up | Chase remaining users, confirm every key user has a second method | No key user left without a backup |

**Support and communications.** Brief the service desk first, then each user individually. The message covers what the key is, why they were chosen for it, how to register it, to keep it safe, and who to call if it is lost. For users affected by the SMS and voice retirement, say plainly that this is the replacement and by when.

## 4. Challenges

| Challenge | How I would handle it |
| --- | --- |
| **Lost or forgotten keys** | A second key or method for each user, a verified recovery route using a Temporary Access Pass, and spare stock. |
| **Key cost and lead time** | Size the stock before the first wave and limit keys to the users who need them. |
| **A key is not in the session** | Microsoft does not count an unavailable device-bound passkey as an alternative, so users need a second method. |
| **Apps or sessions without passkey support** | Test the main apps in the pilot and plan a fallback. |
| **Password reset** | Registering a key does not make a user eligible for self-service password reset, so check they have other eligible methods. |
| **Users unsure about a physical key** | Explain that IT owns it, nothing is installed on their phone, and give a short how-to. |

## 5. Result

This is a design walkthrough, so the outcomes below are what the rollout is built to deliver, not figures from a live deployment.

**Intended outcome:** privileged and regulated roles are on approved device-bound keys, users who cannot use a phone-based method have a working alternative, every key user has a backup method, and nobody is blocked when SMS and voice are retired.

**How I would measure it:** keys registered against the named list, key users with a backup method, tickets and lost-key rate, and sign-in failures after retirement.

## 6. Lessons learned

- **Ask why a user needs a key.** The reason decides whether they get a key, a synced passkey or the Authenticator app.
- **Never issue just one key.** A single key makes every lost key a lockout.
- **Check authentication strengths early.** A user can register a key correctly and still be blocked by a strength that does not allow it.
- **Order stock and write the lost-key process before the first wave.**
- **Keep keys for the people who need them.** For most users the app or a synced passkey is cheaper to run.
- **Use deadlines as a prompt, not the whole plan.** The SMS and voice retirement set the timing, but the same rollout also strengthens the roles that matter most.

---

### References

- [Passkeys (FIDO2) in Microsoft Entra ID (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passkeys-fido2)
- [Passkeys by default and retirement of Microsoft-provided SMS and voice authentication (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sms-voice-retirement)
- [What happens when Microsoft-provided SMS and voice retire (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sms-voice-retirement-experience)
