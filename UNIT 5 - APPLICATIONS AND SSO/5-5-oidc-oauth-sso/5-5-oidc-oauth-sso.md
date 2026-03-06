# OIDC/OAuth SSO — The Modern Approach
*SAML is XML and enterprise. OIDC is JSON and modern. Both get the user signed in, but OIDC is simpler.*

> 🟡 **Intermediate** · Unit 5, Module 5.5 · ⏱️ 55 minutes

---

## What You Will Learn

- Explain the difference between SAML and OIDC SSO at a high level
- Describe the OAuth 2.0 authorization code flow that OIDC uses
- Register a new application for OIDC/OAuth SSO
- Configure redirect URIs and secrets
- Decode an ID token and read its claims
- Choose between SAML and OIDC for a new app integration

---

## Why This Matters

*SC-300: Implement access management for applications — Configure OpenID Connect and OAuth 2.0 single sign-on*

SAML is the enterprise standard for older SaaS (Salesforce, ServiceNow). OIDC is the modern standard for everything new. GitHub, modern Azure applications, and most startups use OIDC. As an Entra ID admin, you'll encounter both, but OIDC is faster to set up and easier to troubleshoot. This module teaches you the OIDC flow and when to choose it over SAML.

---

## 🔄 SAML vs OIDC: Which One and Why

| Aspect | SAML | OIDC |
|---|---|---|
| **Data format** | XML (complex) | JSON (simple) |
| **Protocol foundation** | SAML (proprietary) | OAuth 2.0 (standard) |
| **User verification** | App trusts assertion directly | App verifies with Entra ID |
| **Setup complexity** | More steps, more config | Simpler, faster |
| **Debugging** | Decode XML assertions | Read JSON tokens |
| **When to use** | Enterprise SaaS (Salesforce, Workday) | Modern apps, APIs, custom apps |
| **Common in** | 2010s enterprise | 2020s tech industry |

**Simple rule:**
- **Gallery app requires SAML?** → Use SAML (it's pre-configured)
- **Custom app you're building?** → Use OIDC (it's simpler)
- **Choosing for a new SaaS app?** → Check their docs; most support OIDC now

---

## 🔐 The OAuth 2.0 Authorization Code Flow (OIDC Uses This)

OIDC is built on OAuth 2.0. Here's the flow when Morgan Chen signs in to a custom Contoso app using OIDC:

**Step 1: User clicks "Sign in with Contoso"**
- The app redirects to Entra ID: `https://login.microsoftonline.com/...?client_id=...&redirect_uri=...&scope=openid`
- The app is asking: "Verify this person and send me their ID"

**Step 2: User authenticates to Entra ID**
- Morgan enters credentials (and MFA if required)
- Entra ID verifies Morgan is who they claim to be

**Step 3: Entra ID sends back an authorization code**
- Entra ID redirects back to the app: `https://app.contoso.com/callback?code=ABC123XYZ`
- The authorization code is short-lived (minutes) and single-use

**Step 4: App exchanges code for tokens**
- The app's backend server (not the browser) uses the code to request tokens from Entra ID
- Entra ID responds with an ID token (who the user is) and access token (what they can do)

**Step 5: App grants access**
- The app reads the ID token, sees "Morgan Chen, morgan.chen@contoso.com"
- The app creates a session for Morgan and grants access

**Key difference from SAML:** In OIDC, the app actively asks Entra ID "Is this person really who they claim?" rather than trusting an assertion. This is more flexible and works better with APIs and microservices.

---

## 🎟️ ID Token vs Access Token (OIDC)

Both are JWTs (JSON Web Tokens). You encountered these in Module 3.1.

**ID Token:**
- Proves who the user is (authentication)
- Contains: `oid` (object ID), `email`, `displayname`, `upn`
- Used by: The app (to identify the user)
- Lifetime: 1 hour (short)

**Access Token:**
- Proves what the user can do (authorization)
- Contains: `scp` (scopes), `aud` (audience), permissions
- Used by: APIs that need authorization
- Lifetime: 1 hour (short)

For a simple SSO scenario, only the ID token matters. The app reads it and knows who the user is.

---

## 📋 What You Must Configure for OIDC

Simpler than SAML:

| Element | What it is | Example |
|---|---|---|
| **Client ID** | Unique identifier for your app | `12345678-1234-1234-1234-123456789012` |
| **Redirect URIs** | Where to send the user after sign-in | `https://app.contoso.com/callback` |
| **Client secret** | Password for the app to prove its identity (kept secret) | (Randomly generated, 40+ characters) |
| **Scopes** | What permissions the app requests | `openid` (verify identity) `email` (send email) `profile` (send profile) |

That's it. No certificates, no XML, no attribute mapping. The tokens (ID token, access token) automatically include the user's attributes.

---

## Lab 5.5.1 — Register an App for OIDC SSO

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 5.2 complete (you understand App Registrations)

You'll register a new app configured for OIDC.

1. Navigate to **Applications** → **App registrations** → **+ New registration**.

2. **Name:** `Contoso Intranet Portal`

3. **Supported account types:** **Accounts in this organizational directory only**

4. **Redirect URI (optional):**
   - Platform: **Web**
   - URI: `https://intranetportal.contoso.com/auth/callback`

5. Click **Register**.

6. On the app overview page, copy and note:
   - **Application (client) ID** — You'll share this with developers
   - **Directory (tenant) ID** — Your tenant ID

7. On the left menu, click **Certificates & secrets**.

8. Under **Client secrets**, click **+ New client secret**.

9. **Description:** `Web app secret`

10. **Expires:** Select **6 months** (common practice; longer = more risk)

11. Click **Add**. A secret is generated. Copy it and save it in a secure location. **You can only see this secret once** — if you lose it, you must create a new one.

12. This secret is what the app backend uses to authenticate to Entra ID and exchange authorization codes for tokens.

---

## Lab 5.5.2 — Configure Scopes and Permissions

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.5.1 complete

1. On the **Contoso Intranet Portal** app registration page, click **API permissions** on the left menu.

2. Click **+ Add a permission**.

3. Select **Microsoft Graph**.

4. Select **Delegated permissions** (the app will act on behalf of the signed-in user).

5. Search for and select `email`, `openid`, and `profile` (these are standard OIDC scopes).

6. Click **Add permissions**.

7. You should see three permissions listed:
   - `email` — The app can read the user's email address
   - `openid` — The app can verify the user's identity
   - `profile` — The app can read basic profile info (name, display name, etc.)

8. Click **Grant admin consent for Contoso Ltd**. Consent is granted immediately.

---

## Lab 5.5.3 — Use Graph Explorer to Get an ID Token and Decode It

**Time:** 20 minutes
**Tools:** Graph Explorer (`aka.ms/ge`), jwt.ms
**Prerequisite:** Lab 5.5.2 complete

1. Open Graph Explorer at `aka.ms/ge`.

2. Sign in as **alex.lee@yourdomain.onmicrosoft.com**.

3. At the top of the request panel, look for the **Access token** tab.

4. Click on **Access token** to view Alex's current token.

5. Copy the entire token (it's a long string starting with `eyJ`).

6. Open `jwt.ms` in a new tab.

7. Paste the token and let it decode. You're looking at an access token, which contains:
   - `oid` — Alex's object ID
   - `tid` — Tenant ID
   - `aud` — The audience (who this token is for, usually Microsoft Graph)
   - `scp` — Scopes (what the app is allowed to do)

8. This is what the Contoso Intranet Portal would receive as an access token. The app uses it to call Microsoft Graph on behalf of Alex.

9. For SSO, the app also needs an **ID token**, which is simpler:
   - Contains `oid` (object ID)
   - Contains `email` (email address)
   - Contains `displayname` (display name)
   - Is used *only* for identifying the user, not for API calls

---

## Lab 5.5.4 — Compare OIDC (Simple) vs SAML (Complex)

**Time:** 10 minutes
**Tools:** Text editor
**Prerequisite:** Lab 5.5.3 complete

Create a comparison table in a text file:

**OIDC Setup (what you just did):**
- Create app registration in Azure
- Create client secret
- Configure redirect URI
- Done — no XML, no certificates to configure

**SAML Setup (Module 5.4):**
- Get Entity ID and ACS URL from the vendor
- Configure Basic SAML Configuration in Entra ID
- Download signing certificate
- Configure attribute claims (email, name, etc.)
- Upload certificate to vendor

OIDC is 1/3 the configuration steps. This is why modern apps prefer OIDC.

---

## The Scenario Check

Contoso has registered a new internal application (Contoso Intranet Portal) for OIDC SSO. The app registration has a Client ID and Client Secret stored securely. Scopes are configured: `openid` (identity), `email`, `profile`.

When Alex Lee opens the portal and clicks "Sign in," the portal redirects to Entra ID. Alex signs in. Entra ID issues an authorization code. The portal's backend exchanges the code for an ID token (proving Alex is Alex) and optionally an access token (for calling Microsoft Graph). Alex is now signed in to the portal — all within seconds, no XML parsing required.

---

## Check Your Understanding

- [ ] Explain the difference between SAML and OIDC in one sentence.
- [ ] Describe the four steps of the OAuth 2.0 authorization code flow.
- [ ] In the app registration you created, identify the Client ID, Client Secret, and Redirect URI.
- [ ] Decode an access token in jwt.ms and find the `aud` (audience) claim.
- [ ] State when you would choose OIDC over SAML for a new app integration.

---

## Common Mistakes

**Exposing the client secret in the frontend code.**
The client secret is generated by Entra ID and must be kept secret — shared only between the app's backend and Entra ID. Frontend code (JavaScript, HTML) must *never* include it. If you accidentally commit it to GitHub, immediately rotate it (create a new secret).

**Using the wrong redirect URI.**
You create an app for `https://intranetportal.contoso.com/auth/callback`. The actual app is at `https://app.contoso.com/callback`. Sign-in will fail — Entra ID will reject the redirect because it doesn't match. Always verify redirect URIs match the deployed app exactly.

**Assuming all apps support OIDC.**
You want to use OIDC for everything because it's simpler. The SaaS vendor only supports SAML. You're stuck using SAML. Always check the vendor's documentation first.

**Not securing the client secret.**
You generate a client secret and share it in Slack, email, or GitHub. Anyone with that secret can authenticate as your app and get tokens. Treat client secrets like passwords — store them in secure vaults (Azure Key Vault, HashiCorp Vault, etc.), never in source code.

---

## What's Next

Module 5.6 teaches **User Assignment** — how to control which users and groups can access an app. So far, you've configured SSO (how they sign in) but not access control (who can sign in). Module 5.6 bridges the gap: assignment required, conditional access for apps, and group-based access.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.4 — SAML SSO — Step by Step](../5-4-saml-sso/5-4-saml-sso.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.6 — User Assignment →](../5-6-user-assignment/5-6-user-assignment.md)
