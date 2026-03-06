# Enterprise Applications vs App Registrations
*The same app exists in two places in your tenant. Understanding where each one is and what each one controls is the difference between confused and confident.*

> 🟡 **Intermediate** · Unit 5, Module 5.2 · ⏱️ 55 minutes

---

## What You Will Learn

- Describe what an App Registration is and who creates it
- Describe what an Enterprise Application (Service Principal) is and what it's for
- Register a new application from scratch in App Registrations
- Find the corresponding Enterprise Application created automatically
- Identify which settings are configured on the App Registration side vs the Enterprise Application side
- Explain why admin consent is granted on the Enterprise Application, not the App Registration

---

## Why This Matters

*SC-300: Plan and implement access management for applications — Manage app registrations and service principals*

Module 5.1 introduced the two objects. This module teaches you the *creation flow* and *governance split*. When you configure an app, you'll split time between two admin consoles: App Registrations (developer settings) and Enterprise Applications (admin governance). Confusing where to configure what is the second most common mistake after confusing which object is which.

---

## 🔨 Who Creates What

**App Registration** — Created by a developer (could be you, someone on your team, or the app vendor)
- Happens in **Applications** → **App registrations**
- Defines the technical details: authentication method, required permissions, where the app redirects after sign-in
- One App Registration = one application definition, reused by every tenant using it

**Enterprise Application** — Created automatically when the app is deployed to a tenant
- Happens in **Applications** → **Enterprise applications**
- Defines governance in *your specific tenant*: who can use it, what permissions it's granted, how it authenticates
- One Enterprise Application per tenant, even if multiple tenants use the same app

**Analogy:** App Registration is the blueprint. Enterprise Application is the house you built from that blueprint — your house has your furniture, your rules, your security cameras.

---

## 🔐 The Permission Boundary

Here's where confusion happens:

**Bad assumption:** "I'll grant permissions in the App Registration."

**Reality:** Permissions are granted on the **Enterprise Application** side. The App Registration *requests* them; the Enterprise Application *grants* them per tenant.

| Decision | Lives on | Example |
|---|---|---|
| "I want permission to read all users" | **App Registration** (developer requests this) | Slack developer says "I need `User.Read.All`" |
| "My tenant will grant permission to read all users" | **Enterprise Application** (admin decides yes/no) | Contoso admin clicks **Admin consent** on the Slack Enterprise App |
| "Slack can access Exchange" | **Enterprise Application** (per-tenant permission) | Only Contoso grants this; Acme Corp doesn't |
| "Users must re-consent to scope changes" | **Enterprise Application** (per-tenant policy) | Contoso sets "require consent = yes" |

---

## 📋 Side-by-Side Comparison

| Setting | App Registration | Enterprise Application |
|---|---|---|
| **Redirect URIs** | Defined here (where to send user after auth) | Read-only link to App Reg |
| **Permissions (API access)** | App *requests* these | Tenant *grants* these via admin consent |
| **Admin consent status** | Not shown | Shows **granted** or **not granted** |
| **User assignments** | Not applicable | You assign users/groups here to limit access |
| **Conditional Access** | Not applicable | You target this app with CA policies |
| **Single sign-on** | Not configured here | Configured here (SAML certificate, claims, etc.) |
| **Owner** | Developer/vendor | Tenant admin |
| **Delete consequence** | Breaking change for all tenants using it | Only affects your tenant |

---

## Lab 5.2.1 — Register a New App and Observe Both Sides

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 5.1 complete

You'll register a new application (putting yourself in a developer role), then find the Enterprise Application that was auto-created.

1. Navigate to **Applications** → **App registrations** → **+ New registration**.

2. **Name:** `Contoso Intranet App`

3. **Supported account types:** Select **Accounts in this organizational directory only (Contoso Ltd only - Single tenant)**

4. **Redirect URI (optional):**
   - Platform: **Web**
   - URI: `http://localhost:3000/auth/callback`
   - (This is a local development callback. In production it would be your actual domain.)

5. Click **Register**.

6. You now see the App Registration for "Contoso Intranet App". Note these fields:
   - **Application (client) ID** — The unique identifier for this app
   - **Directory (tenant) ID** — Your tenant ID
   - **Redirect URIs** — The callback you specified

7. Copy the **Application (client) ID** and paste it in a text file. You'll use it to find the Enterprise Application.

---

## Lab 5.2.2 — Find the Enterprise Application and Compare

**Time:** 20 minutes
**Tools:** Entra admin center, Application ID from Lab 5.2.1
**Prerequisite:** Lab 5.2.1 complete

1. Navigate to **Applications** → **Enterprise applications**.

2. Use the search bar to search for `Contoso Intranet App`. Click on it in the results.

3. You're now viewing the Enterprise Application (Service Principal) for the app you just registered.

4. Note these fields:
   - **Object ID** — Unique identifier for this instance in your tenant
   - **Application ID** — Should match the Application (client) ID from the App Registration
   - **Managed** — Shows "No" (because you created it, not Microsoft)

5. Scroll down. Find **Owned by** — shows "Contoso Ltd" (because you created it in your tenant).

6. On the left menu, note the differences from the App Registration:
   - **Users and groups** — Control who can access this app
   - **Properties** — Visibility and access restrictions
   - **Consent and permissions** — Shows what permissions the app has been granted
   - **Conditional Access** — Target CA policies to this app

---

## Lab 5.2.3 — Request and Grant API Permissions

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.2.2 complete

You'll request a permission on the App Registration side, then grant it on the Enterprise Application side.

**Step 1: Request permission in the App Registration**

1. Navigate to **Applications** → **App registrations** → **Contoso Intranet App**.

2. On the left menu, click **API permissions**.

3. Click **+ Add a permission**.

4. In the panel, select **Microsoft Graph**.

5. Select **Delegated permissions** (the app will act on behalf of the signed-in user).

6. Search for and select `User.Read` — allows the app to read the signed-in user's profile.

7. Click **Add permissions**.

8. You should see in the list: **User.Read** — Status shows **No admin consent required** (because User.Read is low-risk and users can self-consent).

9. Click **+ Add a permission** again.

10. Select **Microsoft Graph** → **Application permissions** (the app will act as itself).

11. Search for and select `User.Read.All` — allows the app to read all users in the tenant.

12. Click **Add permissions**.

13. You should see in the list: **User.Read.All** — Status shows **Admin consent required** (because this is a high-permission).

14. Click **Grant admin consent for Contoso Ltd**. A dialog asks to confirm. Click **Yes**.

15. The status changes to **Granted** with today's date.

**Step 2: Verify the permissions appear on the Enterprise Application**

16. Navigate to **Applications** → **Enterprise applications** → **Contoso Intranet App**.

17. On the left menu, click **Permissions** (or **API permissions** depending on your portal version).

18. You should see both permissions you just granted:
    - **User.Read** — Status: **Consented**
    - **User.Read.All** — Status: **Admin consented** (with the date you granted it)

19. This confirms: **Permissions are requested on the App Registration and granted on the Enterprise Application.** Both consents are recorded on the Enterprise app side.

**You'll know this worked when:** You can see the same permissions listed on both the App Registration side (requested) and the Enterprise Application side (granted).

---

## Lab 5.2.4 — Understand the Redirect URI and Authentication Flow

**Time:** 10 minutes
**Tools:** Entra admin center, text editor
**Prerequisite:** Lab 5.2.3 complete

Redirect URIs are a critical part of app security. You'll see how they connect the App Registration to a real authentication flow.

1. Navigate to **Applications** → **App registrations** → **Contoso Intranet App**.

2. On the left menu, click **Authentication**.

3. Under **Redirect URIs**, you should see `http://localhost:3000/auth/callback` that you added earlier.

4. Note the **Supported account types** section — **Single tenant** (only users in Contoso can sign in).

5. Scroll down. Under **Advanced settings** → **Allow public client flows**, you'll see options:
   - This affects what authentication flows the app can use
   - Public clients (like mobile apps, single-page apps) need this enabled

6. Navigate to **Applications** → **Enterprise applications** → **Contoso Intranet App**.

7. On the left menu, click **Properties**.

8. You should see that the visibility and access settings are independent of the App Registration. The Enterprise Application has its own governance layer.

**Why this matters:** If a malicious app tries to authenticate as "Contoso Intranet App", it must redirect to `http://localhost:3000/auth/callback`. If it tries to redirect to `http://attacker.com`, Entra ID rejects it. The Redirect URI is a security boundary — only the legitimate app can complete authentication.

---

## The Scenario Check

Contoso has now registered a new intranet application. An App Registration for "Contoso Intranet App" exists in the App registrations blade. An Enterprise Application (Service Principal) was automatically created in the Enterprise applications blade. The Application ID on both sides matches, confirming they're the same app in different contexts.

Permissions were requested on the App Registration (User.Read, User.Read.All). When Contoso's admin granted consent on the Enterprise Application, the permissions became active for the app to use.

The Redirect URI was set to a local development server. In production, when Contoso developers deploy this app to `https://intranet.contoso.com`, the Redirect URI will be updated to `https://intranet.contoso.com/auth/callback`. Only then can users authenticate through the live app.

---

## Check Your Understanding

- [ ] Create a second test app in App Registrations (`test.app@yourdomain`). Find it in Enterprise applications using the Application ID.
- [ ] From the Enterprise Application, explain which permissions have been granted (if any).
- [ ] Describe where you would go to assign users to limit access to the app.
- [ ] State what would happen if you deleted the Enterprise Application but left the App Registration intact.
- [ ] Explain why Redirect URIs exist and what they protect against.

---

## Common Mistakes

**Trying to grant permissions on the App Registration.**
You see **API permissions** on the App Registration and assume you grant permissions there. You don't — you *request* them there. Granting (admin consent) happens on the Enterprise Application side. If permissions show "No admin consent" on the App Registration, navigate to the Enterprise Application and grant consent there.

**Deleting the Enterprise Application thinking the app still exists.**
You delete the Enterprise Application for Slack. The app is gone from your tenant — users can't access it. The App Registration (Slack's global definition) still exists, but it doesn't matter for your tenant. If you want Slack back, re-add it from the gallery (a new Enterprise Application is created).

**Changing Redirect URIs on the Enterprise Application.**
The Redirect URI must be changed on the **App Registration** side, not the Enterprise Application. The Enterprise app reads it from the App Registration. If you try to configure authentication on the Enterprise Application, you're looking at the wrong place.

**Assuming one app = one App Registration = one Enterprise Application globally.**
This is almost true, but not quite. If two organizations add the same SaaS app (e.g., Slack), there is one Slack App Registration (global) but *two* Slack Enterprise Applications — one per tenant. Each tenant's Enterprise Application is independent.

---

## What's Next

Module 5.3 teaches Single Sign-On (SSO) for gallery apps — the most common use case. You'll take the Slack app you added in Module 5.1, configure basic SSO on the Enterprise Application, and test signing in through it. You'll see the permissions you just granted actually being used.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.1 — The App Integration Big Picture](../5-1-app-integration-big-picture/5-1-app-integration-big-picture.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.3 — Adding a Gallery App and First SSO →](../5-3-adding-gallery-app-sso/5-3-adding-gallery-app-sso.md)
