# Rolling out Temporary Access Pass (TAP) across an organisation

A walkthrough of how I would set up Microsoft Entra Temporary Access Pass (TAP) as the standard way to register and recover sign-in methods for every user, not just admins and guests.

---

## 1. Context

Before writing this case study, I did some research into how many organisations implement TAP. I found it was typically used for only two narrow groups: admin accounts and guest accounts. This makes sense for larger organisations to have it setup this way, as it is a small, well-understood population and allows for very tight control over issuing TAPs.

However, I wanted to document how I would implement TAP within a smaller organisation, across their entire user-base. This changes the problem in three ways:

- **Volume.** Many more TAP requests, mostly new starters and people who have lost a device or MFA method.
- **Attack surface.** A TAP is effectively a credential, so social engineering of admins that can issue one becomes a big risk.
- **Dependencies.** The helpdesk, HR onboarding, device provisioning and Conditional Access all need to fit around it.

## 2. Goal

Replace AD-HOC MFA resets with one verified, time-limited and auditable path for registering and recovering authentication methods.

Specifically, the implementation of TAP will set out to:

- Let new starters register passwordless or MFA methods securely on day one.
- Give users who lose a device or method a recovery route that does not depend on the method they lost.
- Keep the risk of a leaked or socially engineered pass low, especially for privileged accounts.
- Give security a complete audit trail of who issued what, to who, and when.

## 3. Approach

### How TAP works

A TAP is a time-limited passcode that lets a user sign in without a password so they can register a lasting method, such as a passkey, Authenticator passwordless sign-in or Windows Hello for Business.

1. The TAP authentication method policy is enabled for a scoped group.
2. An authorised issuer verifies the user's identity and creates the pass, which is shown once.
3. The issuer delivers it through an agreed secure channel.
4. The user signs in and registers a lasting method.
5. The pass expires, or is used up if it is one-time.

Two important side-notes; each user can only have one TAP at a time (a new pass replaces the old one), and the session obtained with a TAP is limited to the pass's lifetime, so users must finish registration inside that window.

### Policy configuration

The policy supports lifetimes from 10 minutes to 30 days, with a 1 hour default. Individual passes can override the default within the policy's minimum and maximum.

| Setting | Starting point | Reason |
| --- | --- | --- |
| Scope | Pilot group first, then an all-users group; break-glass accounts excluded | Limits blast radius; rollback is removing the group |
| Default lifetime | 1 hour | Matches the Microsoft default and is enough to register a method |
| Maximum lifetime | 8 hours | Caps the exposure of an issued pass; can be altered if passes must be issued ahead of a start date |
| One-time use | Decide after testing | One-time is stronger but can need tighter timing during passwordless registration; multi-use eases longer onboarding |
| Issuer roles | Service desk for standard users; higher role (2nd / 3rd Line) or approval for privileged accounts | Least privilege: whoever can issue a TAP can effectively take over that account |

### Security controls

Most controls sit around issuing TAPs rather than the TAP policy itself.

- **Identity verification before issuing.** One written standard, for example manager approval or ticket plus call-back. Automated identity proofing is a later option.
- **Who can issue.** Service desk for standard users only. Privileged accounts get one-time use, a short lifetime and an approval step. Administrative units can limit the scope of issuers.
- **Delivery channel.** Never to the mailbox the user is locked out of, and never in a ticket others can read. Verbally to a verified user, via the manager, or to a pre-registered phone.
- **Conditional Access.** Protect the "Register security information" user action with a trusted location or compliant device requirement. Check which authentication strengths accept a TAP and review existing policies against that.
- **Monitoring.** Alert on TAP creation (especially for privileged users), several passes for one user, use from unusual locations, and creation outside business hours. Audit logs go to the SIEM.
- **Revoking.** Delete the pass and revoke sessions if a user reports loss or suspected misuse.

### Core user journeys

**New starter**

1. HR confirms the start date and manager.
2. IAM creates the account and adds it to the TAP-scoped group.
3. On or just before day one, the service desk issues a pass and the manager hands it over through the agreed channel.
4. The starter signs in, registers Authenticator or a passkey, and sets up Windows Hello for Business if used.
5. The pass expires; the service desk confirms the registration in the audit log.

**Lost device or MFA method**

1. The user contacts the service desk by a route that does not rely on the lost method.
2. The service desk verifies identity to the agreed standard and logs a ticket.
3. A short-lived pass is issued and delivered through the agreed channel.
4. The user registers a replacement method and removes the lost one.
5. If compromise is suspected, active sessions are revoked.

### Phased rollout

Five phases, each ending at a target that has to be met before the next one starts.

| Phase | What happens | Target to move on |
| --- | --- | --- |
| 1. Design | Policy, roles and controls written up | Design decisions agreed with seniors |
| 2. Test | Policy configured for a test group; every registration flow run end to end, including expiry and reuse of one-time passes; alerts built | All flows pass |
| 3. Pilot | IT and security first, then a friendly business group | Standard Operating Procedure (SOP) signed off, feedback addressed |
| 4. Waves | Expand by department or region, with a gap between waves | Volume and issues reviewed per wave |
| 5. Review | Measure results, tune lifetimes and process, consider retiring weaker fallback methods | n/a |

Because the policy is scoped by group, rollback at any phase is just removing the group from the TAP policy.

### Communications and support readiness

| Audience | What they need to know | When |
| --- | --- | --- |
| All staff | What a TAP is, why it is changing, that it is not a password, never to share it, how to request one, how to recognise a genuine request | 3 to 4 weeks before go-live, reminder at go-live |
| New starters | How and when they receive a TAP, what to do with it, the time window | Onboarding packs and welcome emails |
| Managers | Their role in verification or hand-over, what to tell their teams | 2 to 3 weeks before go-live |
| Service desk | Standard Operating Procedures (SOPs), verification standard, escalation route | Training before the pilot |
| Executives and VIPs | A briefing and a high-touch support route | Before the first wave |
| Security and IAM Teams | Monitoring, alerts and incident response steps | Before the pilot |

The service desk Standard Operating Procedures (SOPs) covers verification steps, how to issue, which lifetime to choose, how to deliver, what to do if a pass fails or expires, and how to escalate suspected fraud.

## 4. Challenges

| Challenge | How I would handle it |
| --- | --- |
| **Social engineering of the service desk.** The main attack path to obtaining a TAP. | Written verification standard, call-back or manager approval, alerts on creation, training and an escalation route. |
| **Interception in delivery.** | Agreed delivery channels, short lifetimes, one-time use where flows allow, and revoke on report. |
| **Privileged accounts.** Taking over an admin through a TAP would be the worst case. | Separate stricter process: higher issuer role, approval step, one-time use, short lifetime, and alerts on privileged creation. |
| **One-time passes failing during passwordless registration.** | Test every real flow (Authenticator, passkey, Windows Hello for Business, Autopilot if used) in the test phase before fixing the one-time setting. |
| **Conditional Access conflicts.** A TAP might not satisfy a policy a user is subject to. | Review policies and authentication strengths in test before the pilot. |
| **Helpdesk overload at go-live.** | Waves with gaps between them, extra capacity in the first weeks, a clear user guide and ticket categories to track volume. |
| **Users who cannot reach a helpdesk in person.** Remote and overseas staff, contractors, VIPs. | Remote verification path, manager hand-over, and a high-touch route for executives. |
| **Phishing that imitates TAP requests.** Attackers will copy the process once staff know it exists. | Staff comms on how genuine requests work, with a reminder after go-live. |
| **Accounts that should not use TAP.** Shared, kiosk and service accounts. | Keep them out of scope and handle them under their own controls. |

## 5. Result

This is a design walkthrough, so the outcomes below are what the rollout is built to deliver and how I would measure it, not figures from a live deployment.

**Intended outcome**

- Every user can register or recover their sign-in methods through one verified, time-limited path.
- No standing credentials for recovery: a pass expires, or is used up, within the configured window.
- Every issuance is logged, attributable and alertable.
- Privileged accounts have a separate, stricter route.
- Any phase can be rolled back by removing a group from the policy.

**How I would measure it**

| Measure | What it tells us |
| --- | --- |
| Registration rate of lasting methods | Whether users complete setup inside the pass window |
| Service desk ticket volume and time to recover | Whether the process is workable at scale |
| Failed or expired passes | Whether lifetime and one-time settings are right |
| Alerts raised on TAP creation and use | Whether monitoring is catching unusual behaviour |
| Any confirmed misuse | Whether verification is holding |

## 6. Lessons learned

What I would do differently, or front-load, next time:

- **Test the registration flows before choosing one-time or multi-use.** I would not fix this setting on principle; I would let the real flows decide.
- **Brief the service desk before staff.** The first people to get requests need a tested Standard Operating Procedure (SOP) before anyone else hears about TAP.
- **Plan for phishing from the start.** Announcing a recovery process invites impersonation of it, so the staff comms should say how a genuine request looks as well as how to make one.
- **Scope by group from day one.** It gives a pilot mechanism and a one-step rollback at the same time.

---

### References

- [Configure Temporary Access Pass to register passwordless authentication methods (Microsoft Learn)](https://learn.microsoft.com/en-ie/Entra/Identity/Authentication/howto-authentication-temporary-access-pass)
