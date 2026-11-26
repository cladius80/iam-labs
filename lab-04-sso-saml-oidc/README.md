# Lab 4: SSO Setup with SAML 2.0 and OpenID Connect — Okta & Microsoft Entra ID

![Okta](https://img.shields.io/badge/Okta-007DC1?style=flat&logo=okta&logoColor=white)
![Microsoft Entra ID](https://img.shields.io/badge/Microsoft%20Entra%20ID-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![SAML 2.0](https://img.shields.io/badge/Protocol-SAML%202.0-orange?style=flat)
![OIDC](https://img.shields.io/badge/Protocol-OpenID%20Connect-green?style=flat)

## Overview

This lab covers Single Sign-On (SSO), one of the most common tasks in an IAM role. I configured custom applications in **Okta** and **Microsoft Entra ID** using the two main SSO protocols, **SAML 2.0** and **OpenID Connect (OIDC)**. I also reviewed the most common SSO errors, what causes them, and how to fix them.

| Protocol | Best for | Format |
|---|---|---|
| **SAML 2.0** | Enterprise apps | XML assertions. Complex but powerful. |
| **OpenID Connect** | Modern web and mobile apps | JSON / JWT tokens. Simpler and lighter. |

---

## Objectives

- [x] Understand SAML assertions and the login flow
- [x] Configure a custom SAML application in Okta
- [x] Understand OIDC tokens and scopes
- [x] Create an OIDC application with proper redirect URIs and security
- [x] Configure a SAML application in Entra ID
- [x] Troubleshoot common SSO errors

---

## Key Terms

| Term | Meaning |
|---|---|
| **IdP (Identity Provider)** | The system that authenticates the user (Okta or Entra ID in this lab) |
| **SP (Service Provider)** | The application the user is signing in to |
| **Entity ID / Audience URI** | Unique name that identifies the SP. Must match exactly on both sides. |
| **ACS URL** | Assertion Consumer Service URL, where the IdP sends the SAML response |
| **NameID** | The user identifier sent in the SAML assertion (e.g., email) |
| **Client ID / Client Secret** | OIDC app credentials. The secret stays server-side only. |
| **Redirect URI** | Where the IdP sends the user back after OIDC sign-in |

---

## Part 1 — Custom SAML 2.0 App in Okta

Used when an app is not in the Okta Integration Network catalog.

**Path:** `Admin Console → Applications → Applications → Create App Integration → SAML 2.0`

### 1.1 General Settings
| Setting | Value |
|---|---|
| App name | `Test SAML App` |
| App logo | Optional |

### 1.2 SAML Settings
| Setting | Value |
|---|---|
| Single sign-on URL | `https://example.com/saml/acs` |
| Audience URI (SP Entity ID) | `https://example.com` |
| Name ID format | `EmailAddress` |
| Application username | `Email` |

### 1.3 Attribute Statements
Mapped user attributes to send in the SAML assertion:

| Name | Value |
|---|---|
| `firstName` | `user.firstName` |
| `lastName` | `user.lastName` |
| `email` | `user.email` |

### 1.4 Finish & Collect IdP Metadata
1. Selected the feedback option and clicked **Finish**.
2. Opened **View Setup Instructions** to get the IdP metadata the app (SP) needs:
   - IdP Single Sign-On URL
   - IdP Entity ID (Issuer)
   - X.509 signing certificate

> 📸 *Screenshot: `screenshots/okta-01-saml-settings.png`*
> 📸 *Screenshot: `screenshots/okta-02-attribute-statements.png`*

---

## Part 2 — OIDC App in Okta

**Path:** `Admin Console → Applications → Applications → Create App Integration → OIDC - OpenID Connect`

### 2.1 Application Type
| Type | Used for |
|---|---|
| **Web Application** | Traditional server-side apps |
| Single-Page App | React, Vue, Angular apps |
| Native Application | Mobile apps |

### 2.2 App Settings
| Setting | Value |
|---|---|
| App name | `Test OIDC App` |
| Sign-in redirect URI | `https://example.com/callback` |
| Sign-out redirect URI | `https://example.com` |

### 2.3 Assignments
Set who can access the app: **Allow everyone** or **Limit access to selected groups**.

### 2.4 Credentials
After saving, Okta generates:
- **Client ID**: shared with the app developers
- **Client Secret**: kept secure and never exposed in frontend code

> ⚠️ **Security note:** Never put the Client Secret in frontend JavaScript. It belongs only in server-side code. For Single-Page Apps, use the **Authorization Code flow with PKCE**, which doesn't need a secret.

> 📸 *Screenshot: `screenshots/okta-03-oidc-app-settings.png`*

---

## Part 3 — SAML App in Microsoft Entra ID

**Path:** `Entra admin center → Identity → Applications → Enterprise applications → + New application → Create your own application`

### 3.1 Create the App
- Name: `Test SAML App`
- Option: **Integrate any other application you don't find in the gallery (Non-gallery)**
- Clicked **Create**

### 3.2 Configure Single Sign-On
Opened **Single sign-on** and selected **SAML**.

### 3.3 Basic SAML Configuration
| Setting | Value |
|---|---|
| Identifier (Entity ID) | `https://example.com` |
| Reply URL (ACS URL) | `https://example.com/saml/acs` |
| Sign on URL | `https://example.com/login` |

### 3.4 Attributes & Claims
Under **Attributes & Claims → Edit**, reviewed and customized the user data sent to the app (e.g., `givenname`, `surname`, `emailaddress`, and the Unique User Identifier / NameID).

### 3.5 SAML Signing Certificate
Under **SAML Certificates**, downloaded **Certificate (Base64)** to share with the app.

> 📸 *Screenshot: `screenshots/entra-01-basic-saml-config.png`*
> 📸 *Screenshot: `screenshots/entra-02-attributes-claims.png`*

---

## Part 4 — Troubleshooting SSO

### Common SAML Errors
| Error | Cause | Fix |
|---|---|---|
| Invalid Audience | Entity ID mismatch | Make sure the Entity ID matches exactly in the IdP and SP |
| Invalid Destination | ACS URL mismatch | Check the ACS URL for typos and trailing slashes |
| Certificate Error | Expired or wrong certificate | Upload the current certificate to the SP |
| Clock Skew | Server times differ | Sync NTP time on both servers |
| User Not Found | NameID doesn't match | Check the NameID format (email vs. username) |

### Common OIDC Errors
| Error | Cause | Fix |
|---|---|---|
| `redirect_uri_mismatch` | Callback URL not registered | Add the exact redirect URI to the app config |
| `invalid_client` | Wrong Client ID or secret | Verify the credentials and check for extra spaces |
| `invalid_scope` | Requesting an unknown scope | Only request scopes the IdP supports |
| `access_denied` | User not assigned to the app | Assign the user or group to the application |

### Debugging Tools
- **SAML Tracer**: browser extension for viewing SAML requests and responses
- **JWT.io**: decode and inspect OIDC tokens
- **Okta System Log**: filter for authentication events
- **Entra Sign-in Logs**: detailed failure reasons for each sign-in

---

## Okta vs. Entra ID — SAML Setup Comparison

| Item | Okta | Microsoft Entra ID |
|---|---|---|
| Where to start | Applications → Create App Integration | Enterprise applications → New application |
| SP identifier | Audience URI (SP Entity ID) | Identifier (Entity ID) |
| Where assertions go | Single sign-on URL | Reply URL (ACS URL) |
| User attributes | Attribute Statements | Attributes & Claims |
| IdP details for the SP | View Setup Instructions | SAML Certificates + Set up section |
| Login troubleshooting | System Log | Sign-in Logs |

---

## Real-World Job Relevance

SSO setup and support is core work for **IAM Engineers, IAM Analysts, and Service Desk** teams:

- **App onboarding:** IAM teams often configure SSO for 10–20+ new applications per quarter.
- **Troubleshooting:** "Can't log in to app X" is a common ticket. Knowing the error-to-cause mapping above speeds up resolution.
- **Certificate management:** SAML signing certificates expire. They need to be rotated before expiration so users aren't locked out.
- **Documentation:** Recording each app's Entity ID, ACS URL, claims, and certificate dates supports disaster recovery and audits.
- **Security:** Protecting Client Secrets, using PKCE for SPAs, and limiting app access to the right groups.

---

## Skills Demonstrated

`SAML 2.0` · `OpenID Connect (OIDC)` · `Single Sign-On` · `Okta Application Integration` · `Entra ID Enterprise Applications` · `Attribute / Claims Mapping` · `X.509 Certificates` · `OAuth 2.0 / PKCE` · `SSO Troubleshooting`

---

## Folder Structure

```
lab-04-sso-saml-oidc/
├── README.md
└── screenshots/
    ├── okta-01-saml-settings.png
    ├── okta-02-attribute-statements.png
    ├── okta-03-oidc-app-settings.png
    ├── entra-01-basic-saml-config.png
    └── entra-02-attributes-claims.png
```

---

*Lab environment: Okta and Microsoft Entra ID test tenants. All apps use placeholder `example.com` URLs.*
