# Retiring SMS and voice MFA: migrating users to Authenticator, passkeys and FIDO2 keys

A walkthrough of how I would move users off SMS and voice authentication in Microsoft Entra ID before Microsoft retires its own SMS and voice delivery from February 2027, with specific plans for users who rely on SMS or voice as their default method, users who only keep it as a backup, and users who have a technical or personal reason not to install the Authenticator app.

---

## 1. Context

This migration is to comply with Microsoft's deadline, not only a security improvement. SMS and voice are the weakest common MFA methods (codes can be phished, intercepted or lost to SIM swapping), and moving off them is worth doing on its own. But the reason it has to happen now is that Microsoft is retiring Microsoft-provided SMS and voice authentication in Entra ID (or at least, making it a pay as you go experience).

| Date | What Microsoft says happens |
| --- | --- |
| **1 February 2027** | Rollout begins of the retirement of Microsoft-provided SMS and voice for all users except Global Administrators and external users. Internal guests are included. |
| 1 July 2027 | Retirement begins for Global Administrators and external users. |

Two significant details from Microsoft:

- 1 February is the **start of a rollout**, not a single moment, and Microsoft says it cannot give the specific date it will reach each tenant. So I would plan to finish before 1 February, not on it.
- After retirement, a user whose **only** MFA method is SMS or voice is **blocked at sign-in** until they register a passkey or the Microsoft Authenticator app. The prompt is blocking, there is no opt-out, and closing the page does not remove it.

Users are spread across multiple situations, handling each of these will present their own difficulties:

- People who use SMS or voice as their **default** method, some with no other method registered at all.
- People who use Authenticator or another method day to day but still have SMS or voice registered as a **secondary** method or recovery route.
- People with a **technical or personal reason** for not installing the Authenticator app on a phone.
- Special cases: guests and external users, Global Administrators, and anything that depends on SMS or voice in an unusual way, such as NPS/RADIUS.

## 2. Goal

Make sure no user is locked out or left without a supported method when SMS and voice retire, and that every user ends up on a method that is stronger than SMS.

Specifically, the change set out to:

- Move users who rely on SMS or voice as their default method onto Microsoft Authenticator, or onto a passkey or FIDO2 security key where Authenticator does not suit them, **before** 1 February 2027.
- Remove the dependency on SMS and voice for users who keep it as a secondary or recovery method, so a lost phone or a retired method does not leave them stranded.
- Give users with a technical or personal objection to the app a real alternative (FIDO / Passkeys), with a clear reason for which option fits which situation.

## 3. Approach

### Step 1: Find and segment the users

Microsoft's guidance starts with identifying who is using SMS and voice. I would run Microsoft's `entra-sms-voice-usage-analyzer` PowerShell script (it needs Global Reader, Authentication Policy Administrator or Security Reader) and combine it with the authentication methods registration and activity reports that can be found in Microsoft Entra. Then I would split the users into cohorts:

| Cohort | Who | Risk after retirement |
| --- | --- | --- |
| **A1. SMS/voice only** | SMS or voice is the only registered method | Blocked at sign-in until they register a passkey |
| **A2. SMS/voice default** | SMS or voice is what they actually sign in with, but another method is registered | Falls back to the other method, but may not be ready for it |
| **B. SMS/voice secondary** | Authenticator or another method is primary; SMS or voice is a backup | Loses the backup and may lose SSPR eligibility |
| **C. Has an objection** | Cannot or will not use the Authenticator app (technical or personal reason) | Needs a FIDO2 key or passkey instead |
| **D. Special cases** | Guests, external users, Global Administrators, NPS/RADIUS users, shared or kiosk accounts | Different dates or different technical constraints |

A user can be in more than one cohort, so I would tag each person with a primary cohort and a note, and keep the list in one place that the service desk can see. Ensuring that the service desk are aware of this deprecation, and do not set new users up with SMS / Voice authentication as that would be creating more work. This could happen if a users phone breaks, and they have to use an older phone that doesn't support the Microsoft Authenticator app.

### Step 2: Plan for each cohort

**Cohort A: SMS or voice is the default (highest priority)**

These users are the real deadline risk, because they use the retiring method every day.

1. **Default path: Microsoft Authenticator push.** It is the lowest-friction change for most people, and a user with a working alternative method is not hit by the blocking passkey prompt.
2. **A1 (SMS/voice only) first.** They are the ones who get blocked. They need a second, non-telephony method registered while SMS still works, so I would target them in the earliest waves with a manager-supported message.
3. **A2 (default but another method exists).** Check the other method actually works for them and is usable in all the places they sign in. Microsoft's own caveat is that registering SMS **and** voice does not count as an alternative, and a device-bound passkey that is not available in the current session does not count either.
4. **Where Authenticator does not suit them,** move them to cohort C (below) rather than leaving them on SMS.

**Cohort B: SMS or voice is a secondary method**

These users will keep working, because they have another method, so the risk is quiet breakage rather than lockout.

1. **Confirm they have two good non-telephony methods,** for example Authenticator plus a passkey or key, so that losing a phone does not mean a helpdesk call.
2. **Expect that existing SMS and voice registrations will show as disabled** in My Sign-Ins, with users unable to edit or add them. This is not something I would try to prevent, so I would warn users and tell them what to expect.
3. **Light-touch communication.** These users need to know their backup is going away and what to set up instead.

**Cohort C: users with a technical or personal reason not to use the app**

I would treat objections as legitimate and record the reason, because the reason decides the replacement. Common reasons:

- No smartphone, or a phone that cannot run the app.
- Unwilling to install a work app on a personal phone.
- Phones not allowed in the work area (secure sites, clinical or manufacturing environments).
- Shared workstations or kiosks where a personal phone is not the right credential.
- Accessibility needs that make a phone prompt hard to use.
- No reliable signal or data, or heavy travel.

Microsoft's own recommendation is passkeys, in two forms: **synced passkeys** (stored in a platform credential manager such as iCloud Keychain or Google Password Manager) and **device-bound passkeys** (stored on one device, for example a FIDO2 hardware security key, Passkey in Microsoft Authenticator, or Entra Passkey on Windows).

| Situation | Best option | Why | Trade-off |
| --- | --- | --- | --- |
| Has a modern personal phone, will not install the Authenticator app | **Synced passkey** (platform credential manager) | No new app, built into the phone, low cost, quick to register | Security depends partly on the cloud provider's protections. No attestation, so you cannot restrict it to approved devices. Recovery depends on that user's own account. |
| No smartphone, or phone not permitted at work | **FIDO2 security key** | Works without a phone, device-bound, phishing-resistant, can be issued and controlled by IT | Hardware cost, training and lost-key recovery add helpdesk load |
| Shared workstation, kiosk or frontline role | **FIDO2 security key** (or device-bound passkey on the device where supported) | The credential belongs to the person, not a phone or a shared account | Needs a physical key per person and a process for lost keys |
| Privileged, regulated or high-assurance role | **FIDO2 security key**, or Passkey in Microsoft Authenticator | Device-bound, supports attestation so you can restrict to approved models | Cost, and Microsoft notes attestation excludes synced passkeys in that profile |
| Mostly on a Windows PC, little phone use | **Device-bound passkey on Windows** where available | No phone needed, tied to the device | Confirm availability and requirements in your tenant before promising it |
| Accessibility needs | Offer a choice between key and synced passkey | The right option depends on what the person can use comfortably | Needs a conversation, not a default |

### Step 3: Handle the special cases

- **Global Administrators:** the formal date is later (1 July 2027), but I would move them to phishing-resistant methods now, as they are the highest-value targets, and not wait for the date.
- **External users:** retired on 1 July 2027. Their experience depends on their home tenant, so I would confirm expectations with the partner organisations, not assume.
- **Internal guests:** included in the 1 February date, so they are in the main waves.
- **NPS/RADIUS (for example VPN):** Microsoft says SMS and voice no longer complete MFA for NPS connections, and NPS does not support passkeys. I would identify these users early and plan a different MFA route for them.
- **Shared and kiosk accounts:** keep them out of the campaign and handle them under their own controls.

### Step 4: Phased rollout

Each phase ends at a gate. I would aim to finish enforcement before 1 February 2027, because Microsoft cannot tell me the exact date it reaches the tenant.

| Phase | What happens | Gate to move on |
| --- | --- | --- |
| 1. Discover | Run the usage analyzer, build cohorts A to D, review policy and Conditional Access | Cohort list agreed, special cases identified |
| 2. Prepare | Passkey profiles, Authenticator settings, service desk standard operating procedure (SOP), key stock if needed | Test group can register every method and sign in with it |
| 3. Pilot | IT and a friendly business group across all cohorts, including at least one FIDO2 key user and one synced passkey user | Standard Operating Procedure (SOP) proven, no unexplained lockouts |
| 4. Waves, cohort A first | A1 then A2, then B and C, by department with a gap between waves | Registration and ticket volume reviewed per wave |
| 5. Clean-up | Chase remaining SMS-only users individually, work through exceptions and any telephony-provider requests | Nobody is SMS-only without a named plan |
| 6. Ready for retirement | Final reminder, check Conditional Access, confirm key stock and recovery process | Zero cohort A1 users, exceptions owned and dated |
| 7. Review | After Microsoft's rollout reaches the tenant, review sign-in failures and tickets, and plan the next step (potentially passkeys for everyone) | n/a |

### Step 5: Communications

The messages need to be honest about why this is happening: **Microsoft is retiring SMS and voice**, and doing it for security reasons as well.

| Audience | What they need to know | When |
| --- | --- | --- |
| Cohort A (SMS/voice default) | SMS and voice stop working from February, what to set up and by when, how to get help. The consequence of doing nothing is being blocked at sign-in. | First wave, then reminders at intervals |
| Cohort B (secondary) | Their backup method is going away, what to register instead. | Later wave, lighter touch |
| Cohort C (objectors) | A named alternative: a security key or a passkey, and why it was chosen for them | Individually, before their wave |
| Managers | Which of their people are affected, how to support them | 2 to 3 weeks before each wave |
| Service desk | Standard Operating Procedure (SOP), verification standard, what to do about key issuance and lost keys | Training before the pilot |
| Executives and VIPs | A briefing and a high-touch support route | Before the first wave |
| Guests and external users | The date that applies to them and what their home tenant needs to do | Early, and separately |

Microsoft publishes end-user message templates, which I would adapt in three phases: awareness, action and reminder. I would also say plainly that nobody will ever ask users to read out a code or number, and that an unexpected prompt should be denied and reported.

## 4. Challenges

| Challenge | How I would handle it |
| --- | --- |
| **The deadline is a rollout, not a date.** Microsoft cannot say when retirement reaches the tenant. | Finish before 1 February, with a buffer, and do not rely on a specific day. |
| **SMS-only users get blocked, not warned.** The passkey prompt is blocking with no opt-out. | Prioritise cohort A1, register a second method while SMS still works for proof-up, and chase individually. |
| **Passkey registered but still blocked.** A custom authentication strength that allows SMS or voice but not FIDO2 keeps users blocked. | Audit authentication strengths in Phase 2 and add FIDO2 where needed. |
| **Disabling passkeys or deleting the retirement profile locks users out.** | Treat both as protected settings and do not use them as a rollback. |
| **NPS/RADIUS cannot do passkeys.** | Find these users in Phase 1 and design a separate route, such as a different MFA mechanism for the VPN. |
| **Apps or sessions that do not support passkeys.** Users see disabled options and guidance. | Test the main apps in the pilot, and provide a fallback such as using another device. |
| **A device-bound passkey is not in the session.** Microsoft does not count it as an alternative. | Make sure users who rely on a key also have a second method for when it is not to hand. |
| **Lost or forgotten security keys.** | Issue a second key to critical users, define a verified recovery route using TAP, and have spare stock. |
| **Synced passkeys and provenance.** No attestation, and the security depends on the cloud provider. | Use them for general users, and use attested device-bound profiles for privileged and regulated roles. |
| **Personal objections about privacy or cost.** | Offer a key that IT owns, so no one has to put work software or credentials on a personal phone, and explain what the app does and does not access. |
| **People tempted to weaken policy to avoid the problem.** | Microsoft's guidance is not to weaken Conditional Access or turn off MFA as a workaround, so I would hold that line. |
| **Genuine need to keep SMS or voice.** | Assess the documented regulatory or operational reason, and consider a telephony provider from the Microsoft Security Store as a last resort. |
| **Helpdesk spike in each wave.** | Waves with gaps between them, extra capacity, and a short, tested guide for each method. |

## 5. Result

This is a design walkthrough, so the outcomes below are what the migration is built to deliver and how I would measure it, not figures from a live deployment.

**Intended outcome**

- No user is blocked at sign-in or unable to reset a password when Microsoft's SMS and voice retirement reaches the tenant.
- Users who relied on SMS or voice as their default method are on Authenticator, a passkey or a FIDO2 security key.
- Users who kept SMS or voice as a backup have two working non-telephony methods.
- Users with a technical or personal objection have a named alternative that fits their situation.
- Privileged and regulated roles are on attested, device-bound methods.
- The exception list is small, owned and dated.

**How I would measure it**

| Measure | What it tells us |
| --- | --- |
| Users still SMS-only (cohort A1), tracked weekly | The real deadline risk |
| Users with SMS or voice as default, by wave | How much of the main migration is done |
| Users with two non-telephony methods | Whether backup users are protected |
| Registrations by method (Authenticator, synced passkey, FIDO2 key) | Whether the alternatives are being used |
| Service desk tickets and time to resolve | Whether the process is workable at scale |
| Size and age of the exception list | Whether SMS and voice are genuinely being retired |
| Sign-in failures after the retirement reaches the tenant | Whether anyone was missed |

## 6. Lessons learned

What I would do differently, or front-load, next time:

- **Segment before messaging.** A single message to everyone would have been wrong for each group. SMS-only users need urgency, backup users need a light nudge, and objectors need an individual conversation.
- **Treat objections as inputs.** Asking why someone will not use the app produces the right replacement (key, passkey or provider). It also stops the same objection coming back as a ticket later.
- **Audit authentication strengths and NPS early.** These are the two places where a user can do everything right and still be blocked or fail.
- **Order the stock.** Security keys have lead times and need a recovery process, so I would size the stock and write the lost-key procedure before the first wave.
- **Plan the next step.** Moving off SMS is a floor, not the finish line, so I would use this migration to put passkeys in place for everyone as the longer-term direction.

---

### References

- [Passkeys by default and retirement of Microsoft-provided SMS and voice authentication (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sms-voice-retirement)
- [What happens when Microsoft-provided SMS and voice retire (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sms-voice-retirement-experience)
- [Passkeys (FIDO2) in Microsoft Entra ID (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passkeys-fido2)
- [Run a registration campaign to set up Microsoft Authenticator (Microsoft Learn)](https://learn.microsoft.com/en-gb/entra/identity/authentication/how-to-mfa-registration-campaign)
- [Migrate MFA and SSPR policy settings to the Authentication methods policy (Microsoft Learn)](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-authentication-methods-manage)
- [entra-sms-voice-usage-analyzer (Microsoft, GitHub)](https://github.com/microsoft/entra-sms-voice-usage-analyzer)
