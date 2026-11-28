# Lab 5: MFA Configuration in Okta

![Okta](https://img.shields.io/badge/Okta-007DC1?style=flat&logo=okta&logoColor=white)
![MFA](https://img.shields.io/badge/Focus-Multi--Factor%20Authentication-red?style=flat)
![Okta Verify](https://img.shields.io/badge/Authenticator-Okta%20Verify-blue?style=flat)

## Overview

This lab covers setting up **Multi-Factor Authentication (MFA)** in Okta from start to finish. I enabled authenticators, built an **enrollment policy** that controls which factors users enroll, and built an **authentication (sign-on) policy** that controls when MFA is required. Then I walked through the user enrollment experience, tested the MFA challenge with a test user, and confirmed the events in the Okta System Log.

```
Enable Factors  →  Create Enrollment Policy  →  Create Sign-On Policy  →  Users Enroll  →  Validate
```

> **In plain terms:** MFA is like a bank vault that needs two different keys. A password is one key, and a phone or fingerprint is the other. A thief needs both, so having just one is useless.

---

## Objectives

- [x] Enable and configure MFA authenticators in Okta
- [x] Create an MFA enrollment policy
- [x] Create an authentication policy that requires MFA at sign-in
- [x] Understand the end-user enrollment experience
- [x] Validate MFA with a test login and review the System Log

---

## Background — Authentication Factors

| Factor Type | What It Means | Examples |
|---|---|---|
| Something you **know** | Information in your head | Password, PIN, security questions |
| Something you **have** | A physical item you possess | Phone, hardware key, smart card |
| Something you **are** | Your biometrics | Fingerprint, face, voice |

### Strong vs. Weak MFA Methods

| ✅ Stronger | ⚠️ Weaker |
|---|---|
| FIDO2 / WebAuthn security keys | SMS codes (vulnerable to SIM swapping) |
| Authenticator apps (TOTP) | Email codes (at risk if the email account is compromised) |
| Push notifications (Okta Verify) | Security questions (often guessable) |
| Biometrics (Windows Hello, Face ID) | |

---

## Part 1 — Enable MFA Authenticators

**Path:** `Admin Console → Security → Authenticators`

1. Reviewed the available authenticators:

   | Authenticator | Method |
   |---|---|
   | Okta Verify | Push notifications and TOTP |
   | Google Authenticator | TOTP codes |
   | SMS Authentication | Text message codes |
   | Security Key or Biometric | FIDO2 / WebAuthn |

2. Used **Add Authenticator** for any option not already listed.
3. Configured **Okta Verify** (`Actions → Edit`):
   - Enabled **Push notification**
   - Enabled **TOTP** (6-digit codes)
   - Optionally enabled **Number challenge** for phishing resistance
4. Clicked **Save**.

> 💡 **Best practice:** Enable more than one authenticator type so users have a backup. If someone loses their phone, they can still use a security key.

> 📸 *Screenshot: `screenshots/01-authenticators.png`*

---

## Part 2 — MFA Enrollment Policy

Enrollment policies control **which** factors users must or may enroll.

**Path:** `Security → Authenticators → Enrollment → Add a Policy`

| Setting | Value |
|---|---|
| Policy name | `All Users MFA Policy` |
| Assigned to groups | `Everyone` |

**Factor enrollment options:**

| Option | Meaning |
|---|---|
| **Required** | The user must enroll this factor |
| **Optional** | The user may enroll if they want |
| **Disabled** | The factor isn't available |

**Configuration:** Set **Okta Verify** or **Security Key** to *Required* and the others to *Optional*, then clicked **Create Policy**.

> 📸 *Screenshot: `screenshots/02-enrollment-policy.png`*

---

## Part 3 — Authentication Policy Requiring MFA

Sign-on (authentication) policies control **when** MFA is required, such as every login, a new device, or a risky location.

**Path:** `Security → Authentication Policies → Add a policy`

| Setting | Value |
|---|---|
| Policy name | `Require MFA for All Apps` |

**Rule configuration (`Add rule`):**

| Rule Setting | Value |
|---|---|
| Rule name | `Always require MFA` |
| IF User is | Any user type |
| AND Device is | Any device |
| THEN Access is | Allowed after MFA challenge |
| Re-authentication frequency | Every sign-in attempt |

- Under **Applications**, added the apps this policy applies to.
- Saved and confirmed the policy is **Active**.

> ℹ️ **Policy priority:** Okta checks policies in order, and the first matching policy applies. More specific policies should be placed higher.

> 📸 *Screenshot: `screenshots/03-authentication-policy-rule.png`*

---

## Part 4 — User MFA Enrollment Experience

What an end user sees the first time MFA is required:

1. The user signs in to the Okta dashboard with their username and password.
2. If MFA is required and they haven't enrolled, they see **"Set up multifactor authentication."**
3. The user picks an authenticator (Okta Verify, Security Key, etc.).

**Okta Verify enrollment:**
1. Download the Okta Verify app on the phone.
2. Open the app and tap **Add Account**.
3. Scan the QR code shown on screen.
4. The account is added, and future logins send a push notification.

**TOTP (Google Authenticator) enrollment:**
1. Open the authenticator app.
2. Scan the QR code.
3. Enter the 6-digit code to verify.
4. Future logins require the current code.

**Admin view:** `Directory → People → [User] → More Actions → View all factors`

> 📸 *Screenshot: `screenshots/04-user-enrollment.png`*

---

## Part 5 — Validation

### Test the MFA Flow
1. Opened an **Incognito/Private** window to simulate a new session.
2. Went to the Okta org URL (`your-org.okta.com`).
3. Signed in with a test user's username and password.
4. Confirmed the **MFA challenge** appeared (push notification / code entry).
5. Completed MFA and reached the dashboard.

### Check the System Log
**Path:** `Reports → System Log`

- Filtered by event type **`user.authentication.auth_via_mfa`** (or searched "mfa").
- Reviewed the event details:
  - Who authenticated
  - Which factor was used
  - Success or failure
  - IP address and location

> 📸 *Screenshots: `screenshots/05-mfa-challenge.png`, `screenshots/06-system-log-mfa-event.png`*

---

## Troubleshooting — Common Problems & Fixes

| Problem | Likely Cause | Fix |
|---|---|---|
| User not getting the MFA prompt | Sign-on policy not assigned or ranked too low | Check the policy's assignment and priority order |
| Okta Verify push not received | Phone notifications off or no internet | Check the phone's notification settings and connectivity |
| User locked out after losing phone | Enrolled device unavailable | Admin resets the user's MFA factors, and the user re-enrolls |

---

## Real-World Job Relevance

MFA administration is a daily part of **IAM Engineer, IAM Analyst, and Service Desk** work:

- **MFA mandates:** Rolling out organization-wide MFA requirements
- **Risk-based access:** Setting different MFA requirements for different risk levels (device, network, location)
- **Ticket support:** Troubleshooting "MFA not working" and "not receiving push" tickets
- **Factor resets:** Resetting MFA for users who lose or replace their phones, after verifying their identity
- **Compliance reporting:** Reporting on MFA enrollment rates for audits and security reviews

---

## Skills Demonstrated

`Multi-Factor Authentication` · `Okta Authenticators` · `Okta Verify (Push / TOTP / Number Challenge)` · `FIDO2 / WebAuthn` · `Enrollment Policies` · `Authentication Policies` · `Policy Priority` · `System Log Analysis` · `MFA Troubleshooting`

---

## Folder Structure

```
lab-05-mfa-okta/
├── README.md
└── screenshots/
    ├── 01-authenticators.png
    ├── 02-enrollment-policy.png
    ├── 03-authentication-policy-rule.png
    ├── 04-user-enrollment.png
    ├── 05-mfa-challenge.png
    └── 06-system-log-mfa-event.png
```

---

*Lab environment: Okta test tenant. All identities are fictitious test accounts.*
