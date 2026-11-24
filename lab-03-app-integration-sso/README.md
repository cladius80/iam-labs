# Lab 3: Application Integration & SAML SSO — Okta & Microsoft Entra ID

![Okta](https://img.shields.io/badge/Okta-007DC1?style=flat&logo=okta&logoColor=white)
![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![SAML](https://img.shields.io/badge/Protocol-SAML%202.0-orange?style=flat)
![IAM](https://img.shields.io/badge/Focus-Identity%20%26%20Access%20Management-6f42c1?style=flat)

## Overview

This lab covers how an organization connects business applications such as **Zoom** to its identity provider (IdP), so users sign in once and reach their apps without separate passwords. In both **Okta** and **Microsoft Entra ID**, I added an application from the platform's app catalog, configured **SAML single sign-on**, and assigned access through users and groups. I also reviewed how **SCIM provisioning** automates creating and removing accounts in connected apps.

```
Add App from Catalog  →  Configure SAML SSO  →  Assign Users & Groups  →  (SCIM) Automate Provisioning
```

---

## Lab Scenario

| Item | Value |
|---|---|
| **Application** | **Zoom** (added from the Okta Integration Network and the Entra app gallery) |
| **SSO Protocol** | SAML 2.0 |
| **Assigned User** | Alex Thompson (from [Lab 1](../lab-01-iam-fundamentals/)) |
| **Assigned Group** | `Engineering-Team` |
| **Platforms** | Okta Admin Console, Microsoft Entra admin center |

**Business scenario:** The Engineering team needs access to **Zoom** for meetings. Rather than give each person a separate username and password, IT connects the app to the company's identity provider. Users sign in once with their corporate identity, and access is granted through group membership.

---

## Objectives

- [x] Add an application from the Okta and Entra ID app catalogs
- [x] Configure SAML single sign-on settings
- [x] Assign users and groups to the application
- [x] Understand SCIM provisioning concepts

---

## Key Concepts

| Term | Meaning |
|---|---|
| **Identity Provider (IdP)** | The system that verifies who the user is: Okta or Entra ID in this lab |
| **Service Provider (SP)** | The application the user wants to reach |
| **SAML assertion** | A signed XML message from the IdP telling the SP who the user is |
| **Entity ID / Identifier** | The unique name the IdP and SP use to recognize each other |
| **ACS URL (Reply URL)** | The SP address where the IdP sends the SAML assertion |
| **IdP metadata / certificate** | The IdP's sign-in URL and signing certificate, which the SP uses to trust the assertion |
| **SCIM** | An open standard (System for Cross-domain Identity Management) for automatically creating, updating, and deactivating user accounts in apps |

**SAML sign-in flow (SP-initiated):**

```
User opens app  →  App redirects to IdP  →  User signs in (MFA)  →  IdP sends signed SAML assertion  →  App grants access
```

---

## Part 1 — Okta

### 1.1 Add the Application from the Catalog
**Path:** `Admin Console → Applications → Applications → Browse App Catalog`

1. Searched the Okta Integration Network (OIN) for **Zoom**.
2. Selected it and clicked **Add Integration**.
3. Entered the general settings (application label and the Zoom vanity URL subdomain, e.g. `<company>.zoom.us`) and saved.

> 📸 *Screenshot: `screenshots/okta-01-app-catalog.png`*

### 1.2 Configure SAML Single Sign-On
**Path:** `Applications → Zoom → Sign On tab`

1. Selected **SAML 2.0** as the sign-on method.
2. Entered or confirmed the SP details required by the app (such as the ACS URL and Audience URI / Entity ID).
3. Set the **Application username format** (e.g., Okta username or email).
4. Reviewed the **IdP metadata** and signing certificate that the application uses to trust Okta.

> 📸 *Screenshot: `screenshots/okta-02-saml-settings.png`*

### 1.3 Assign Users and Groups
**Path:** `Applications → Zoom → Assignments tab → Assign`

1. Assigned **Alex Thompson** directly (**Assign to People**).
2. Assigned the **`Engineering-Team`** group (**Assign to Groups**).
3. Confirmed both assignments appear on the Assignments tab.

> **Note:** Group assignment is preferred over individual assignment. New team members get access automatically when they join the group, and lose it when they leave.

> 📸 *Screenshot: `screenshots/okta-03-assignments.png`*

---

## Part 2 — Microsoft Entra ID

### 2.1 Add the Application from the Gallery
**Path:** `Entra admin center → Identity → Applications → Enterprise applications → New application`

1. Searched the **Microsoft Entra application gallery** for **Zoom**.
2. Selected it and clicked **Create**.
3. Confirmed the app appears under **Enterprise applications**.

> 📸 *Screenshot: `screenshots/entra-01-app-gallery.png`*

### 2.2 Configure SAML Single Sign-On
**Path:** `Enterprise applications → Zoom → Single sign-on → SAML`

1. **Basic SAML Configuration:** entered the **Identifier (Entity ID)** and **Reply URL (ACS URL)**.
2. **Attributes & Claims:** reviewed the claims sent to the app (Unique User Identifier = `user.userprincipalname`, plus name and email).
3. **SAML Certificates:** reviewed the signing certificate and Federation Metadata XML.
4. **Set up Zoom:** noted the Login URL and Microsoft Entra Identifier that the app needs.

> 📸 *Screenshot: `screenshots/entra-02-saml-settings.png`*

### 2.3 Assign Users and Groups
**Path:** `Enterprise applications → Zoom → Users and groups → Add user/group`

1. Added **Alex Thompson** and the **`Engineering-Team`** group.
2. Selected the app role (default access) and clicked **Assign**.
3. Confirmed both appear on the **Users and groups** page.

> **Note:** Assigning groups to enterprise apps requires Microsoft Entra ID P1 or P2. On a free tenant, users can be assigned individually.

> 📸 *Screenshot: `screenshots/entra-03-assignments.png`*

---

## Part 3 — SCIM Provisioning (Concepts)

SSO controls **how users sign in**. Provisioning controls **whether their account exists in the app** in the first place. SCIM automates that second part.

| Without SCIM | With SCIM |
|---|---|
| Admin creates each account in the app by hand | Account is created automatically when the user is assigned |
| Profile changes must be updated in two places | Changes in the IdP sync to the app |
| Departing users may keep app accounts (orphaned accounts) | Account is deactivated automatically when access is removed |

**How it works:** the IdP acts as the SCIM client and calls the app's SCIM API (using a base URL and an API token) to create, update, and deactivate users and groups.

| | Okta | Microsoft Entra ID |
|---|---|---|
| **Where it's set up** | `Zoom → Provisioning tab → Integration` | `Enterprise app → Provisioning` |
| **Connection details** | SCIM base URL + API token | Tenant URL + secret token |
| **Actions** | Create Users, Update User Attributes, Deactivate Users | Provision on assignment, update, deprovision on removal |
| **Sync timing** | Triggered by assignment changes | Provisioning cycles about every 40 minutes |

---

## What I Learned

- **Application onboarding** connects SaaS apps to the identity provider, so access is managed in one place.
- **SSO** eliminates password fatigue: users sign in once and reach every assigned app.
- **SCIM provisioning** automates the user account lifecycle in connected apps: create, update, and deactivate.
- **Always assign apps to groups, not individual users.** Access then follows group membership and stays easy to audit.
- **Validate the setup** with SSO tests (Okta: sign in as the user from the End-User Dashboard; Entra: **Test this application** on the SAML page) and by reviewing provisioning logs.

---

## Okta vs. Entra ID — Quick Comparison

| Task | Okta | Microsoft Entra ID |
|---|---|---|
| App catalog | Okta Integration Network (Browse App Catalog) | Microsoft Entra app gallery |
| Where apps live | Applications | Enterprise applications |
| SAML setup | Sign On tab | Single sign-on → SAML |
| SP settings | ACS URL, Audience URI | Reply URL, Identifier (Entity ID) |
| User/group access | Assignments tab | Users and groups |
| Provisioning | Provisioning tab | Provisioning blade |

---

## Real-World Job Relevance

These tasks come up regularly in **IAM Analyst, Identity Engineer, Service Desk, and Systems Administrator** roles:

- **App onboarding:** Connecting new SaaS applications to SSO is one of the most common IAM requests.
- **Fewer passwords, stronger security:** SSO lets one sign-in (with MFA) protect many apps, and reduces password-reset tickets.
- **Access requests:** Granting app access through group assignments keeps access consistent and easy to audit.
- **SSO troubleshooting:** Knowing the Entity ID, ACS URL, claims, and certificate is the basis for fixing failed SAML sign-ins.
- **Lifecycle automation:** SCIM ties onboarding and offboarding to app accounts, closing the orphaned-account gap that auditors look for.

---

## Skills Demonstrated

`SAML 2.0 SSO` · `Application Integration` · `Okta Integration Network` · `Entra Enterprise Applications` · `App Assignment (Users & Groups)` · `Federation Metadata & Certificates` · `SCIM Provisioning Concepts` · `Okta Administration` · `Microsoft Entra ID Administration`

---

## Repository Structure

```
lab-03-app-integration-sso/
├── README.md
└── screenshots/
    ├── okta-01-app-catalog.png
    ├── okta-02-saml-settings.png
    ├── okta-03-assignments.png
    ├── entra-01-app-gallery.png
    ├── entra-02-saml-settings.png
    └── entra-03-assignments.png
```

---

*Lab environment: Okta and Microsoft Entra ID test tenants. All identities are fictitious test accounts.*
