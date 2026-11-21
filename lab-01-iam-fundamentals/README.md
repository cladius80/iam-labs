# Lab 1: IAM Fundamentals — Okta & Microsoft Entra ID

![Okta](https://img.shields.io/badge/Okta-007DC1?style=flat&logo=okta&logoColor=white)
![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![IAM](https://img.shields.io/badge/Focus-Identity%20%26%20Access%20Management-6f42c1?style=flat)

## Overview

This lab covers the core identity lifecycle tasks an IAM or Service Desk analyst performs every day. I completed them in two leading identity platforms, **Okta** and **Microsoft Entra ID**. Using a test identity, I worked through the full flow of onboarding a user, granting access through group membership, supporting a password reset, and offboarding by disabling the account. I then confirmed each change in the audit logs.

```
Create Identity  →  Assign to Groups  →  Grant App Access  →  Monitor & Manage
```

---

## Lab Scenario

| Item | Value |
|---|---|
| **Test User** | Alex Thompson |
| **Job Title** | Software Developer |
| **Department** | Engineering |
| **Security Group** | `Engineering-Team` |
| **Platforms** | Okta Admin Console, Microsoft Entra admin center |

**Business scenario:** Alex Thompson is a new Software Developer joining the Engineering department. He needs an account and membership in the Engineering team group so he gets the right access. Later he needs a password reset, and at the end of his engagement his account must be disabled.

---

## Objectives

- [x] Create a user identity in Okta and Entra ID
- [x] Create a security group and assign members
- [x] Reset a user's password
- [x] Disable a user account (offboarding)
- [x] Validate all changes through system/audit logs

---

## Part 1 — Okta

### 1.1 Create the User
**Path:** `Admin Console → Directory → People → Add Person`

1. Entered user details:
   - First name: `Alex`
   - Last name: `Thompson`
   - Username / Primary email: `alex.thompson@<your-domain>`
   - Title: `Software Developer`
2. Set the password option (admin-set or user-activated).
3. Saved and confirmed the user appeared with an **Active** (or **Pending user action**) status.

> 📸 *Screenshot: `screenshots/okta-01-create-user.png`*

### 1.2 Create the Group & Assign the User
**Path:** `Directory → Groups → Add Group`

1. Created group **`Engineering-Team`** with a description (e.g., *"Engineering department — software developers"*).
2. Opened the group, selected **Assign people**, and added **Alex Thompson**.
3. Confirmed Alex is listed as a group member.

> 📸 *Screenshot: `screenshots/okta-02-group-membership.png`*

### 1.3 Reset Password
**Path:** `Directory → People → Alex Thompson → More Actions → Reset Password`

1. Started a password reset for Alex.
2. Confirmed the reset was issued (email link or temporary password).

> 📸 *Screenshot: `screenshots/okta-03-reset-password.png`*

### 1.4 Disable the Account
**Path:** `Directory → People → Alex Thompson → More Actions → Deactivate` *(or Suspend)*

1. Changed the account to the disabled state.
2. Confirmed the status changed to **Deactivated / Suspended**.

> **Note:** In Okta, **Suspend** temporarily blocks sign-in while keeping assignments. **Deactivate** removes app access and is the standard offboarding action.

> 📸 *Screenshot: `screenshots/okta-04-disable-user.png`*

---

## Part 2 — Microsoft Entra ID

### 2.1 Create the User
**Path:** `Entra admin center → Identity → Users → All users → New user → Create new user`

1. Entered user details:
   - User principal name: `alex.thompson@<your-tenant>.onmicrosoft.com`
   - Display name: `Alex Thompson`
   - Job title: `Software Developer`
   - Department: `Engineering`
2. Recorded the auto-generated initial password.
3. Created the user and confirmed it appears in **All users**.

> 📸 *Screenshot: `screenshots/entra-01-create-user.png`*

### 2.2 Create the Security Group & Assign the User
**Path:** `Identity → Groups → All groups → New group`

1. Group type: **Security**
2. Group name: **`Engineering-Team`**
3. Membership type: **Assigned**
4. Added **Alex Thompson** as a member and created the group.
5. Checked membership under `Groups → Engineering-Team → Members`.

> 📸 *Screenshot: `screenshots/entra-02-group-membership.png`*

### 2.3 Reset Password
**Path:** `Users → Alex Thompson → Reset password`

1. Generated a temporary password.
2. Confirmed the user must change it at next sign-in.

> 📸 *Screenshot: `screenshots/entra-03-reset-password.png`*

### 2.4 Disable the Account
**Path:** `Users → Alex Thompson → Edit properties → Settings → Account enabled` *(uncheck)*

1. Turned off **Account enabled** and saved.
2. Confirmed the user's sign-in is blocked.

> 📸 *Screenshot: `screenshots/entra-04-disable-user.png`*

---

## Part 3 — Validation via Logs

Every identity change should leave an audit trail. I confirmed each action in both platforms:

| Action | Okta — System Log | Entra ID — Audit Logs |
|---|---|---|
| User created | `user.lifecycle.create` | `Add user` |
| Added to group | `group.user_membership.add` | `Add member to group` |
| Password reset | `user.account.reset_password` | `Reset user password` |
| Account disabled | `user.lifecycle.deactivate` / `user.lifecycle.suspend` | `Update user` (AccountEnabled = false) |

**Okta path:** `Reports → System Log` (filter by target user *Alex Thompson*)
**Entra path:** `Identity → Monitoring & health → Audit logs` (filter by target *Alex Thompson*)

> 📸 *Screenshots: `screenshots/okta-05-system-log.png`, `screenshots/entra-05-audit-log.png`*

---

## Okta vs. Entra ID — Quick Comparison

| Task | Okta | Microsoft Entra ID |
|---|---|---|
| Create user | Directory → People → Add Person | Users → New user |
| Group type used | Okta group | Security group (Assigned) |
| Password reset | More Actions → Reset Password | Reset password (temp password) |
| Disable account | Suspend / Deactivate | Account enabled = No |
| Audit trail | System Log | Audit Logs |

---

## Real-World Job Relevance

These tasks map straight to day-to-day work in **Service Desk, IAM Analyst, and Systems Administrator** roles:

- **Onboarding (Joiner):** Provisioning accounts and access for new hires on day one
- **Role-based access (Mover):** Using group membership instead of per-user grants. This keeps access consistent and easy to audit, and follows least privilege.
- **Password support:** One of the most common Tier 1/Tier 2 tickets
- **Offboarding (Leaver):** Disabling accounts promptly is a key security control, because orphaned accounts are a common audit finding.
- **Audit & compliance:** Log evidence of who changed what and when supports SOX, SOC 2, and similar audit requirements.

---

## Skills Demonstrated

`Identity Lifecycle Management (JML)` · `User Provisioning` · `Group-Based Access Control` · `Password Management` · `Account Deprovisioning` · `Audit Log Review` · `Okta Administration` · `Microsoft Entra ID Administration`

---

## Repository Structure

```
lab-01-iam-fundamentals/
├── README.md
└── screenshots/
    ├── okta-01-create-user.png
    ├── okta-02-group-membership.png
    ├── okta-03-reset-password.png
    ├── okta-04-disable-user.png
    ├── okta-05-system-log.png
    ├── entra-01-create-user.png
    ├── entra-02-group-membership.png
    ├── entra-03-reset-password.png
    ├── entra-04-disable-user.png
    └── entra-05-audit-log.png
```

---

*Lab environment: Okta and Microsoft Entra ID test tenants. All identities are fictitious test accounts.*
