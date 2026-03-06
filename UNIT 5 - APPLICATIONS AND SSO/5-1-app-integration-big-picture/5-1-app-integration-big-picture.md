# The App Integration Big Picture
*Before you configure a single app, understand what two objects you're creating and why you need both.*

> 🟢 **Beginner** · Unit 5, Module 5.1

---

## What You Will Learn

- Explain what it means to "connect an app to Entra ID" and why it matters
- Distinguish between pre-integrated gallery apps and custom/non-gallery apps
- Identify the two objects created by every app integration and what each one is for
- Navigate to both the App Registration and the Enterprise Application for the same app
- Explain why Contoso needs both objects and what happens if you configure one but not the other

---

## Why This Matters

*SC-300: Implement access management for applications — Plan and implement service principal objects for application access*

Every identity challenge in this series so far has been about users: creating them, managing them, applying policies to them. Starting with Unit 5, the focus shifts to applications. SaaS apps (Salesforce, Slack, Jira), custom applications built by your developers, automation scripts — all of these need to authenticate to Entra ID and access Microsoft 365 resources on behalf of users.

Connecting an app to Entra ID is not a single action. It creates two separate objects with different responsibilities. Confusing these two objects is the #1 mistake in app integration. By the end of this module, you'll understand what each one is and why Entra ID makes you manage them separately.

---

## 🔌 What Does "Connect an App to Entra ID" Mean?

When Contoso integrates Slack with Entra ID, here's what happens:

1. **Slack (the vendor) publishes an integration** in the Microsoft Entra application gallery, a catalog of pre-integrated SaaS apps
2. **Contoso (the tenant admin) "adds" Slack** from the gallery
3. **Two objects are created automatically in Contoso's tenant:**
   - An **App Registration** — the technical definition of the app
   - An **Enterprise Application (Service Principal)** — Contoso's local instance and governance configuration

The app is now "connected" — Slack can ask Entra ID "Is this person allowed to use my app?" and Entra ID can answer.

---

## 🏛️ The Two Objects: App Registration vs Enterprise Application

Every app integration creates two objects. They are related but not the same.

| | **App Registration** | **Enterprise Application** |
|---|---|---|
| **What is it?** | Global app definition (created by the developer) | Local instance in your tenant (created when you add the app) |
| **Who owns it?** | The app vendor (Slack, Salesforce, etc.) | Your organization (Contoso) |
| **Where is it?** | Registered in the gallery once (shared across all tenants using it) | **One per tenant** — your tenant has its own |
| **What lives here?** | Redirect URIs, authentication protocols, required permissions | Admin consent, user assignments, single sign-on config, conditional access |
| **What do you change?** | Usually nothing — vendors manage this | Everything — this is where your governance happens |
| **Example path in portal** | **Applications** → **App registrations** | **Applications** → **Enterprise applications** |

---

## 🔑 Why Two Objects? The Permission Problem

**Scenario: Slack wants to read Contoso's users**

Slack says: "I want to read the list of users in this Entra ID tenant so I can match them to Slack workspaces."

**On the App Registration side (global definition):**
- Slack requests permission: "Grant me `User.Read.All`" (read all users)
- This request exists in Slack's app definition — every tenant sees it

**On the Enterprise Application side (Contoso's instance):**
- Contoso's admin must *decide*: "Do I allow Slack to read all my users?"
- Contoso clicks **Admin consent** on the Enterprise Application
- The decision is stored in Contoso's instance only — doesn't affect other tenants

Another tenant, Acme Corp, might say "No, Slack cannot read all our users." Their Enterprise Application for Slack has no consent. Same app, two different governance decisions.

This separation lets vendors publish one app globally and let each organization govern it independently.

---

## 🎨 Gallery Apps vs Custom Apps

| Type | Definition | Admin Console Path | When You Use It |
|---|---|---|---|
| **Gallery app** | Pre-integrated by Microsoft, ready to add in minutes | **Enterprise applications** → search or browse the gallery | Any SaaS app in the Microsoft Entra gallery (Slack, Salesforce, Okta, etc.) |
| **Custom app** | Built by your organization, not in the gallery | **App registrations** → create new | Your own internal application, custom SaaS not in the gallery |

For gallery apps, you don't create the App Registration yourself — it already exists. You just add it, and the Enterprise Application is created automatically.

For custom apps, your developer creates the App Registration first (with redirect URIs and required permissions), then it appears in Enterprise Applications when deployed.

---

## Lab 5.1.1 — Browse the Microsoft Entra Application Gallery

**Tools:** Entra admin center
**Prerequisite:** Module 1.1 (developer tenant set up)

1. Navigate to **Applications** → **Enterprise applications**.

2. Click the **+ New application** button at the top.

3. The gallery opens. You're looking at hundreds of pre-integrated SaaS applications. Browse the categories on the left:
   - **Featured** — Most popular apps
   - **Categories** — Browse by use case (HR, Security, Development, etc.)
   - **A-Z** — Alphabetical list

4. Search for three apps relevant to a typical organization:
   - Search for `Slack` — note that it's marked **In the gallery**
   - Search for `Salesforce` — find both the main Salesforce app and Salesforce sandbox version
   - Search for `GitHub` — note whether it's in the gallery

5. Click on one app (e.g., **Slack**). A preview panel opens showing:
   - App name and logo
   - Description
   - **Create** button
   - Note: clicking **Create** would add it to your tenant and auto-create the Enterprise Application

6. Do not create it yet — just note the structure. Click **Cancel** or click outside the panel.

**You'll know this worked when** you can browse the gallery and find at least three SaaS apps you recognize.

---

## Lab 5.1.2 — Find an Enterprise Application and Explore Its Properties

**Tools:** Entra admin center
**Prerequisite:** Lab 5.1.1 complete

Every Microsoft 365 service (Teams, SharePoint, Exchange) is already integrated as an Enterprise Application. You'll explore one now.

1. Navigate to **Applications** → **Enterprise applications**.

2. Use the search bar to find **Teams**. Click on **Microsoft Teams** in the results.

3. The Teams Enterprise Application page opens. Note these sections on the left menu:
   - **Overview** — Basic info about the app
   - **Users and groups** — Which users/groups can access this app
   - **Single sign-on** — SSO configuration
   - **Properties** — App metadata
   - **Conditional Access** — CA policies targeting this app

4. Click **Overview**. You'll see:
   - **Application ID** — A GUID that uniquely identifies this app in Entra ID
   - **Object ID** — Unique identifier for this Enterprise Application in your tenant
   - Tenant information
   - **Managed** — Yes (because it's a Microsoft app)

5. Scroll down. Find the **Managed application** section. It shows:
   - **App registration** link — Click this link to jump to the corresponding App Registration

6. Click the **App registration** link. You're now viewing the **App Registration** side of Teams.

7. On the App Registration page, note:
   - **Application (client) ID** — Same GUID you saw on the Enterprise Application side
   - **Tenant information**
   - **Redirect URIs** — Where Teams redirects after authentication
   - **Certificates & secrets** — Used for app authentication (not visible for managed Microsoft apps)

8. At the top of the page, find **Managed by Microsoft** label — indicating this is a Microsoft-owned app.

9. Use the browser back button or search to return to **Enterprise applications** → **Teams**. You've now navigated both directions: Enterprise Application ↔ App Registration.

**You'll know this worked when** you can:
- Navigate from an Enterprise Application to its App Registration
- Identify the Application ID that links them
- State what information lives on each side

---

## Lab 5.1.3 — Compare Two Apps: a Gallery App vs a Microsoft Service

**Tools:** Entra admin center
**Prerequisite:** Lab 5.1.2 complete

You'll add a gallery app and compare its structure to the Microsoft Teams app you explored in Lab 5.1.2.

**Step 1: Add a gallery app**

1. Navigate to **Applications** → **Enterprise applications** → **+ New application**.

2. Search for and find **Slack** in the gallery.

3. Click **Slack**. A side panel shows the app details with a **Create** button.

4. Click **Create**. Entra ID creates:
   - An Enterprise Application for Slack in your tenant
   - An underlying Service Principal object
   - Automatic sign-in is not yet configured (you'll do this in Module 5.3)

5. You should see: "Slack has been created successfully."

**Step 2: Compare Slack (gallery app you just added) to Teams (Microsoft service)**

6. Navigate to **Applications** → **Enterprise applications**.

7. Search for **Slack**. Click on the Slack app you just created.

8. On the Slack Enterprise Application page, note:
   - **Managed** — Likely shows "No" or "External" (because Slack is managed by Slack Inc., not Microsoft)
   - **Application ID** — A GUID unique to this app
   - **Object ID** — Unique ID of this instance in your tenant

9. Scroll down and look for the **App registration** link. Click it to jump to Slack's App Registration.

10. On the Slack App Registration page, note:
    - It's owned by Slack Inc. (the vendor)
    - The Application ID matches the one on the Enterprise Application side
    - Redirect URIs are already configured by Slack

11. Return to **Enterprise applications** → **Slack**. Compare to **Teams**:

| Attribute | Teams | Slack |
|---|---|---|
| Managed by | Microsoft | Slack Inc. (vendor) |
| Added to your tenant by | Default (Microsoft) | You (via gallery) |
| SSO configuration | Pre-configured | You must configure |
| User assignment required? | No | You can enforce it |

**You'll know this worked when** you understand that both Teams and Slack have an Enterprise Application and an App Registration, but Teams is managed by Microsoft while Slack is managed by the vendor.

---

## The Scenario Check

Contoso has now added Slack to its tenant. An Enterprise Application for Slack exists, and the underlying Service Principal is created. Slack is not yet configured for single sign-on — that happens in Module 5.3. Alex Lee and Morgan Chen both have Slack accounts, but Entra ID does not yet know about them. The Slack Enterprise Application shows "No users assigned yet" because Contoso hasn't explicitly assigned users (optional for apps like Slack that allow direct sign-up).

Contoso also explored Teams and confirmed it's a pre-integrated Microsoft service that's automatically managed.

---

## Check Your Understanding

- [ ] Explain the difference between an App Registration and an Enterprise Application without looking at this module
- [ ] Navigate to **Enterprise applications** and find **Slack** (that you just added) and **Teams** — confirm both exist
- [ ] From the Slack Enterprise Application, click the link to its App Registration — verify the Application ID matches
- [ ] State why Entra ID creates two objects instead of one combined object
- [ ] In the Slack Enterprise Application, find the **Application ID** — this links it to the App Registration

---

## Common Mistakes

**Confusing App Registration with Enterprise Application.**
You're asked to "configure SSO for an app." You navigate to **App registrations** and try to configure it there. SSO is configured on the **Enterprise Application** side, not the App Registration. The App Registration defines *what* the app is; the Enterprise Application defines *how your organization uses it*.

**Thinking you can delete the App Registration to remove an app.**
You added an app to your tenant (created an Enterprise Application), later you decide to remove it, and you delete the App Registration. This breaks things for every tenant using that app. If you added it from the gallery, delete the Enterprise Application (the **app** in your tenant), not the App Registration (the global definition).

**Assuming all apps are in the gallery.**
You have a custom internal application. You search the gallery for it. It's not there. This is correct — only SaaS vendors publish gallery apps. For custom apps, the developer creates an App Registration in your tenant directly, and it automatically appears in Enterprise applications.

**Not giving admin consent.**
A gallery app asks for permissions (e.g., "read all users"). You add it to your tenant but forget to click **Admin consent**. Users cannot use the app because consent was never granted. Permission requests require admin consent.

---

## What's Next

Module 5.2 dives deeper into the App Registration vs Enterprise Application distinction. You'll register a new app from scratch (putting you in developer shoes), see the App Registration first, then watch the Enterprise Application appear automatically. You'll understand the creation flow and what data lives where.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 4.8 — Production CA Policy Stack](../../UNIT%204%20-%20CONDITIONAL%20ACCESS/4-8-production-ca-policy-stack/4-8-production-ca-policy-stack.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.2 — Enterprise Applications vs App Registrations →](../5-2-enterprise-applications-vs-app-registrations/5-2-enterprise-applications-vs-app-registrations.md)
