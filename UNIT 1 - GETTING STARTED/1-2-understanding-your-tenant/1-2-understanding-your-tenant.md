# Understanding Your Tenant
*You know what a tenant contains, where to find its ID, and which decisions you cannot undo.*

> 🟢 **Beginner** · Unit 1, Module 1.2

---

## What You Will Learn

- Describe what a Microsoft Entra ID tenant is and what it contains
- Find your Tenant ID and primary domain in the admin center
- Explain the difference between an `.onmicrosoft.com` domain and a custom domain
- Identify the tenant-level settings a Global Administrator controls
- State three things about a tenant that cannot be changed after creation

## Why This Matters

Every configuration you do in this series lives inside your tenant. Understanding what a tenant is and how it is structured prevents mistakes that are expensive to fix later. The `.onmicrosoft.com` domain assigned at creation stays with your tenant permanently. The Tenant ID is how every Microsoft system and third-party app identifies your organization. Get these concepts wrong and they cause problems for years.

---

## 🏢 One Organization. One Tenant. No Exceptions.

A tenant is a dedicated, isolated instance of Microsoft Entra ID that belongs exclusively to your organization. Nothing inside it is shared with or visible to other organizations by default.

Everything your organization has in Entra ID lives inside this one container:

| What lives in your tenant | Examples |
|---|---|
| Identities | Users, guests, service principals, managed identities |
| Groups | Security groups, Microsoft 365 groups, dynamic groups |
| Applications | Enterprise apps, app registrations |
| Devices | Entra-joined, Entra-registered, Hybrid-joined |
| Policies | Conditional Access, authentication methods, PIM settings |
| Logs | Sign-in logs, audit logs, provisioning logs |

When you sign in to `entra.microsoft.com`, you are always inside exactly one tenant. The tenant you are in is shown in the top-right corner of the admin center next to your username.

Most organizations have one tenant. Large enterprises sometimes have multiple (for subsidiaries or test environments), but each tenant is completely separate. There is no such thing as a sub-tenant or a parent tenant.

---

## 🆔 The Two Things That Identify Your Tenant

### Tenant ID

Your Tenant ID is a GUID (globally unique identifier) — a string of 32 hexadecimal characters formatted like this:

```
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Example: `a1b2c3d4-e5f6-7890-abcd-ef1234567890`

This is how Microsoft's backend systems, third-party applications, and APIs identify your organization. When a SaaS app asks for your "Tenant ID" or "Directory ID" during SSO setup, this is what it means. You will use it constantly.

### Primary Domain

Every tenant is assigned a `.onmicrosoft.com` domain at creation — for example, `contoso.onmicrosoft.com`. This is your tenant's permanent domain. It is used in:

- User principal names that have not been assigned a custom domain (`alex@contoso.onmicrosoft.com`)
- The default Reply URL for app registrations
- SharePoint URLs (`contoso.sharepoint.com`)

> ⚠️ **Warning:** The `.onmicrosoft.com` domain cannot be renamed, removed, or transferred. Whatever name was chosen when the tenant was provisioned is permanent. If you provisioned a test tenant as `randomtest2024.onmicrosoft.com`, that name is attached to the tenant forever.

### Custom Domains

A custom domain is a domain you own — for example, `contoso.com` — that you add to your tenant and verify via a DNS record. Once verified, you can assign user UPNs on that domain (`alex@contoso.com` instead of `alex@contoso.onmicrosoft.com`).

Adding a custom domain does not remove the `.onmicrosoft.com` domain. Both exist permanently once the tenant is created. The `.onmicrosoft.com` domain is the fallback if a custom domain is removed.

---

## ⚠️ Three Things You Cannot Change After Day One

**1. The `.onmicrosoft.com` domain name.**
Set at tenant creation. Permanent. You can add custom domains, but you cannot rename or remove the `.onmicrosoft.com` domain. Choose carefully when creating production tenants.

**2. The Tenant ID.**
The GUID assigned to your tenant never changes. It identifies your organization in every Microsoft system, in every log, and in every third-party integration. It cannot be reassigned.

**3. Tenants cannot be merged.**
If Contoso Ltd acquires Fabrikam Ltd — two separate Entra ID tenants — those tenants cannot be combined into one. Users, groups, apps, and policies cannot be merged. You can synchronize identities between tenants using B2B or cross-tenant sync, but each tenant remains a separate container. Plan your tenant structure before you need to change it.

---

## Lab 1.2.1 — Find Your Tenant ID and Primary Domain

**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Module 1.1 completed. Signed in as your sandbox admin account.

1. Sign in to `entra.microsoft.com` with your sandbox admin account.

2. In the left navigation menu, click **Identity**, then click **Overview**. The Overview page displays your tenant's basic information.

3. On the Overview page, locate and write down:
   - **Tenant ID** — shown under "Basic information." It is the GUID that identifies your organization.
   - **Primary domain** — your `.onmicrosoft.com` domain, also shown under "Basic information."
   - **Display name** — the organization name you set when you created the tenant.

4. Click the **copy icon** next to the Tenant ID. Open Notepad or any text editor and paste it. You will need this value throughout the series when configuring app integrations and SSO.

5. Now find the Tenant ID a second way. Click the **Settings icon** (gear) at the top of the admin center. Select **Tenant properties** from the menu. Confirm the same Tenant ID appears here.

> 💡 **Tip:** The Tenant ID also appears in the URL when you navigate to certain admin pages. Look for a URL segment in the format `/[your-tenant-id]/` in the browser address bar.

**You'll know this worked when** you have your Tenant ID written down and confirmed from two different locations in the admin center.

---

## Lab 1.2.2 — Explore Tenant-Level Settings

**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Lab 1.2.1 completed.

Tenant-level settings are configurations that apply to the entire organization — not to a single user, group, or app. A Global Administrator controls them. This lab explores the most important ones.

1. In the left navigation menu, click the **Settings icon** (gear) at the top, then select **Tenant properties**. Review the fields:
   - **Name** — the display name of your organization in Microsoft services
   - **Country or region** — set at creation, determines your data residency
   - **Technical contact** — the email address Microsoft sends important notifications to
   - **Global privacy contact** and **Privacy statement URL** — optional but required by some compliance frameworks

   > ⚠️ **Warning:** The country or region field affects where Microsoft stores your data. It cannot be changed after tenant creation.

2. In the left navigation menu, expand **Identity**, then click **Users**. At the top of the Users section, click **User settings**.

   Review these settings:
   - **App registrations** — "Users can register applications." If set to **Yes**, any user in your tenant can create app registrations. If set to **No**, only admins can. Note the current value.
   - **LinkedIn account connections** — controls whether users can connect their LinkedIn profiles to Microsoft apps. Note the current value.
   - **Show keep user signed in** — controls whether the "Stay signed in?" prompt appears at sign-in.

3. In the left navigation menu, expand **Identity**, then click **External Identities**. Click **External collaboration settings**. Review:
   - **Guest user access** — controls what guests can see in your directory
   - **Guest invite settings** — controls who can send B2B invitations
   - **Collaboration restrictions** — allows or denies invitations to specific domains

   You will configure these in Module 2.8. For now, note the defaults.

4. In the left navigation menu, expand **Protection**, then click **Password protection**. Review:
   - **Custom banned passwords** — you can add organization-specific words (your company name, product names) that users cannot use as passwords
   - **Lockout threshold** and **Lockout duration** — how many failed attempts before the account is locked and for how long

**You'll know this worked when** you can state from memory where to find User settings, External collaboration settings, and Password protection in the admin center.

---

## Lab 1.2.3 — Add a Custom Domain (Optional)

**Tools:** Entra admin center (`entra.microsoft.com`), access to a domain's DNS settings
**Prerequisite:** A domain you own with access to its DNS records. Skip this lab if you do not own a domain — the concept is what matters.

> 🔑 **Key concept:** Adding a custom domain requires you to prove you own it by creating a TXT record in your domain's DNS settings. Entra ID checks for that record before activating the domain.

1. In the left navigation menu, expand **Identity**, then click **Settings**, then click **Domain names**.

   You will see your `.onmicrosoft.com` domain listed as **Primary**. This domain cannot be removed.

2. Click **+ Add custom domain** at the top of the page.

3. In the panel that opens, enter your domain name (for example, `yourdomain.com`) and click **Add domain**.

4. Entra ID will display a TXT record to add to your DNS. It will look like:

   | Type | Name | Value | TTL |
   |---|---|---|---|
   | TXT | @ | MS=ms12345678 | 3600 |

5. Log in to your domain registrar or DNS provider (GoDaddy, Namecheap, Cloudflare, etc.). Add the TXT record exactly as shown. DNS propagation typically takes 5–30 minutes but can take up to 48 hours.

6. Return to Entra ID and click **Verify**. If DNS has propagated, Entra ID will verify the domain and mark it as **Verified**.

7. Once verified, you can set this domain as the **Primary** domain by selecting it and clicking **Make primary**. New users created after this point will default to having UPNs on this domain.

**If you cannot complete this lab:** Navigate to **Identity → Settings → Domain names** and observe the current state. Note that the `.onmicrosoft.com` domain is listed as **Primary** and has no **Remove** option. That absence is the point — it is permanent.

**You'll know this worked when** your custom domain appears in the Domain names list with a status of **Verified**.

---

## The Scenario Check

Contoso Ltd's tenant is examined and understood. The Tenant ID has been recorded — this value will be used when connecting SSO apps in Unit 5, configuring app registrations in Unit 11, and troubleshooting sign-in logs throughout the series. The `.onmicrosoft.com` domain is confirmed as permanent. No Contoso users have been created yet — that begins in Module 2.1. The tenant-level settings (User settings, External collaboration settings, Password protection) are at their defaults and will be modified in later modules as each topic is introduced.

---

## Check Your Understanding

- [ ] Open `entra.microsoft.com` and find your Tenant ID without reading these instructions. Write it from memory.
- [ ] State the difference between the `.onmicrosoft.com` domain and a custom domain in one sentence.
- [ ] Explain why two organizations cannot merge their Entra ID tenants, even if one company acquires the other.
- [ ] Navigate to User settings from memory and state what the "App registrations" setting controls.
- [ ] Navigate to Domain names from memory and confirm how many domains are currently in your tenant.
- [ ] State the three things about a tenant that cannot be changed after creation.

---

## Common Mistakes

**Treating the developer sandbox the same as a production tenant.**
The developer sandbox has a randomly generated `.onmicrosoft.com` domain name. That name is permanent. If you ever build a production environment, create the tenant with a planned name from the start — not a throwaway name you intend to rename later.

**Confusing Tenant ID with Object ID.**
Both are GUIDs. The Tenant ID identifies the organization. The Object ID identifies a specific object inside the tenant (a user, a group, an application). When an app or a support ticket asks for the Tenant ID, it means the organization's GUID — not the ID of your admin account.

**Assuming adding a custom domain removes the `.onmicrosoft.com` domain.**
It does not. Both exist permanently once created. If you remove a custom domain from a tenant, every user on that domain loses their UPN and must be reassigned to the `.onmicrosoft.com` domain or another verified domain. Removing a custom domain affects users — adding one does not.

---

## What's Next

Module 1.3 covers the Entra admin center in detail — how it is organized, where every major feature lives, and how to navigate it in under 30 seconds. You have already visited five sections of the admin center in this module. Module 1.3 maps the entire workspace and builds the navigation habit that every lab in this series depends on.

---

> 📚 **Entra ID — Zero to Admin** · Unit 1: Getting Started
>
> [← Module 1.1 — What Is Microsoft Entra ID?](../1-1-what-is-entra-id/1-1-what-is-entra-id.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 1.3 — The Entra Admin Center →](../1-3-entra-admin-center/1-3-entra-admin-center.md)
