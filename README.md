# Entra Authentication Methods
 
Practical guides and case studies on modernising authentication in Microsoft Entra ID, moving away from weak, phishable methods towards passwordless and phishing-resistant sign-in.
 
> **Note:** These solutions were designed and built outside of work, in my personal Microsoft Azure tenant, and are intended as practical guides for other administrators. They are not based on, and do not describe, any employer's environment or configuration.
 
---
 
## Why this matters
 
Passwords and legacy MFA methods such as SMS and voice calls are still the most common way into an organisation. They can be phished, intercepted or bypassed through SIM-swapping, MFA fatigue and man-in-the-middle attacks.
 
Modern authentication strategy is about:
 
- **Removing weak methods** such as SMS and voice from everyday use
- **Strengthening the default** with Microsoft Authenticator (number matching and additional context)
- **Onboarding securely** without ever handing out a password, using Temporary Access Pass
- **Reaching phishing resistance** with FIDO2 security keys, passkeys and Windows Hello for Business
- **Enforcing it** through the Authentication Methods Policy and Conditional Access authentication strengths
Each case study in this repository covers one step on that journey.
 
---
 
## Case studies
 
| Document | Topic | Summary |
| --- | --- | --- |
| [`temporary-access-pass-rollout`](https://github.com/ejneedham00/Entra-Authentication-Methods/blob/main/Temporary%20Access%20Pass%20Rollout) | Temporary Access Pass | Rolling out TAP for passwordless onboarding and account recovery |
| [`sms-to-authenticator-migration`](https://github.com/ejneedham00/Entra-Authentication-Methods/blob/main/SMS%20to%20Authenticator%20App%20Migration) | SMS → Authenticator | Phasing out SMS and voice in favour of Microsoft Authenticator |
| [`fido2-security-keys`](https://github.com/ejneedham00/Entra-Authentication-Methods/blob/main/FIDO%20Key%20Utilisation) | FIDO2 security keys | Deploying hardware keys for phishing-resistant sign-in |
 
More case studies will be added over time.
 
---
 
## Case study structure
 
Each folder follows the same format so they're easy to compare:
 
1. **Context** – the scenario and starting point
2. **Goal** – what the change set out to achieve
3. **Approach** – planning, configuration and phased rollout
4. **Challenges** – edge cases, user impact and how they were handled
5. **Result** – the outcome
6. **Lessons learned** – what I'd do differently next time
7. **Scripts** – supporting Microsoft Graph PowerShell scripts, where relevant
---
 
## Disclaimer
 
This repository is my own personal documentation, shared in the hope that it helps other administrators. It is not official Microsoft guidance. Always check the current Microsoft documentation first, as features and defaults change, and use this material as a supplement rather than a source of truth. Scripts may also be referenced. If used, please test in a non-production tenant before using them.
