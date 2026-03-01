# The Entra Admin Center — Your Workspace
*You can navigate to any Entra ID feature in under 30 seconds, with or without the search bar.*

> 🟢 **Beginner** · Unit 1, Module 1.3 · ⏱️ 35 minutes

---

## What You Will Learn

- Identify the six top-level sections of the Entra admin center and what each one contains
- Explain the difference between the Entra admin center and the Microsoft 365 admin center
- Navigate to any feature in this series using both the left nav and the search bar
- Pin your most-used sections to the Favorites bar
- Build a reference of direct URLs for the sections you will use every day

## Why This Matters

Every lab in this series takes place in the Entra admin center. Spending 30 seconds hunting for a setting every time you need it adds up. Admins who know the workspace navigate confidently. Admins who don't, click around and make mistakes. This module turns the admin center into familiar territory before anything is configured.

---

## 🗺️ Six Sections. Each One Has a Job.

The left navigation menu in the Entra admin center (`entra.microsoft.com`) is divided into six top-level sections. Each section groups features by purpose.

| Section | What lives here | Used for |
|---|---|---|
| **Identity** | Users, Groups, External Identities, Roles & admins, App registrations | Managing who exists in your tenant |
| **Protection** | Conditional Access, Authentication methods, Identity Protection, PIM, Password protection | Securing how identities access resources |
| **Governance** | Access reviews, Entitlement management, Lifecycle workflows, Terms of use | Managing who has access over time |
| **Applications** | Enterprise applications, App registrations | Connecting and managing apps |
| **Devices** | All devices, Device settings | Managing device identity and state |
| **Workload identities** | Service principals, managed identities | Non-human identities for apps and automation |

Microsoft reorganizes the admin center without announcement. When a setting moves, use the search bar at the top to find it by name rather than navigating the menu blindly.

**What's under each section you will use most:**

**Identity:**
- Users → All users, Sign-in logs, Audit logs, Provisioning logs
- Groups → All groups, Licenses
- External Identities → External collaboration settings, Cross-tenant access settings
- Roles & admins → All roles, Administrative units

**Protection:**
- Conditional Access → Policies, Named locations, What If tool, Authentication strengths
- Authentication methods → Policies, Password reset (SSPR)
- Identity Protection → Risky sign-ins, Risky users, Risk detections
- Privileged Identity Management → My roles, Pending requests, Manage

**Governance:**
- Access reviews
- Entitlement management → Access packages, Catalogs
- Lifecycle workflows

---

## 🔄 Two Admin Centers. They Are Not the Same Thing.

Every Microsoft 365 organization has two admin centers. Beginners confuse them constantly.

**Entra admin center** (`entra.microsoft.com`) — identity-focused. This is where you work for everything in this series: Conditional Access, MFA, PIM, Identity Protection, app integrations, governance. If the task involves identity, access, or security policy, it is done here.

**Microsoft 365 admin center** (`admin.microsoft.com`) — service-focused. This is where you manage Microsoft 365 subscriptions, billing, Exchange settings, Teams settings, and SharePoint administration. License assignment is also cleaner here when managing large numbers of users.

Both admin centers can create and edit users. The Entra admin center is the authoritative source for identity attributes and policies — the M365 admin center surfaces a simplified view of the same underlying user objects.

| Task | Where to do it |
|---|---|
| Create a user | Either — prefer Entra admin center for identity work |
| Assign a Microsoft 365 license | Microsoft 365 admin center |
| Create a Conditional Access policy | Entra admin center only |
| Configure PIM | Entra admin center only |
| Manage a shared mailbox | Microsoft 365 admin center (Exchange) |
| Configure SSPR | Entra admin center only |
| View sign-in logs | Entra admin center only |
| Manage billing and subscriptions | Microsoft 365 admin center only |

> 💡 **Tip:** From the Microsoft 365 admin center, you can click **Identity** in the left nav to jump directly to the Entra admin center. Many admins land in M365 first and navigate from there.

---

## ⚡ Two Ways to Find Anything in Under 30 Seconds

**The left nav** is for features you visit regularly. You learn the structure once, then navigate by memory.

**The search bar** (at the top of every page, shortcut: click the magnifying glass or press `/`) searches across features, users, groups, applications, and devices simultaneously. Type any partial name and results appear instantly.

The search bar finds:
- Admin center sections by name (type `conditional access` to jump directly to that blade)
- Users by display name or UPN (type `alex` to find Alex Lee)
- Groups by name
- Enterprise applications by name
- App registrations by name

You do not need to be on a specific page to use it. Search works from anywhere in the admin center.

---

## Lab 1.3.1 — Navigate Every Major Section

**Time:** 15 minutes
**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Module 1.1 completed. Signed in as your sandbox admin account.

Navigate to each section listed below using the left nav. For each one, note one thing you observe about what is already there (pre-existing data from the sandbox, or an empty state). Do not change any settings.

1. **Identity → Users → All users** — How many users are in your tenant? What columns are shown by default?

2. **Identity → Groups → All groups** — How many groups exist? Note the Group type column. What types are shown?

3. **Identity → External Identities → External collaboration settings** — What is the current guest invite setting? (Do not change it.)

4. **Identity → Roles & admins → All roles** — How many built-in roles are listed? Find the **Global Administrator** role in the list and click on it. How many assignments does it currently have?

5. **Protection → Conditional Access** — How many policies exist? The answer should be zero if you are using a fresh instant sandbox.

6. **Protection → Authentication methods → Policies** — Which authentication methods are currently enabled?

7. **Protection → Identity Protection → Risky users** — Is the list empty? (It should be.)

8. **Protection → Privileged Identity Management** — Click **Get started** or **Quick start** if prompted. You will configure PIM in Unit 7 — for now, just confirm it is accessible.

9. **Governance → Access reviews** — Is the list empty?

10. **Governance → Entitlement management → Access packages** — Is the list empty?

11. **Applications → Enterprise applications → All applications** — How many applications are listed? The sandbox comes with pre-integrated Microsoft services.

12. **Devices → All devices** — Are any devices registered? (Likely none unless you registered a device in Module 1.1.)

**You'll know this worked when** you can navigate to all 12 locations above without getting lost or needing to re-read the list.

---

## Lab 1.3.2 — Use Search to Navigate

**Time:** 10 minutes
**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Lab 1.3.1 completed.

1. Click the **search icon** (magnifying glass) at the top of the admin center. A search bar appears across the top of the page.

2. Type `sign-in logs`. Select **Sign-in logs** from the results under **Services**. You are now on the Sign-in logs page. Note: you did not navigate through the left menu.

3. Click the search icon again. This time type `Test User`. You should see **Test User 01** appear under **Users** (created in Module 1.1). Click on **Test User 01**. You are now on that user's profile page.

4. Click the search icon again. Type `conditional`. Select **Conditional Access** from the results. You are on the Conditional Access blade.

5. Click the search icon again. Type `pim`. Select **Privileged Identity Management** from the results.

6. Click the search icon again. Type `authentication methods`. Select **Authentication methods** from the results.

**You'll know this worked when** you can reach Sign-in logs, Conditional Access, PIM, and Authentication methods in under 10 seconds each using only the search bar.

---

## Lab 1.3.3 — Pin Favorites and Set Up Bookmarks

**Time:** 10 minutes
**Tools:** Entra admin center (`entra.microsoft.com`), web browser
**Prerequisite:** Lab 1.3.2 completed.

The Entra admin center has a **Favorites** bar — a customizable row of pinned sections that appears at the top of the left navigation. Pin the sections you will use most.

1. In the left navigation menu, hover over **Users** under the **Identity** section. A **star icon** appears to the left of the item. Click the star to pin Users to your Favorites.

2. Pin the following sections the same way:
   - **Conditional Access** (under Protection)
   - **Sign-in logs** (under Identity → Users → Sign-in logs — hover over the item in the submenu to see the star)
   - **Enterprise applications** (under Applications)
   - **Privileged Identity Management** (under Protection)

3. Scroll to the top of the left navigation. You should now see a **Favorites** section with your pinned items listed. Click each one to confirm the pin worked.

4. Now set up browser bookmarks for the pages you will open constantly. In your browser, navigate to each URL below and bookmark it:

   | Page | URL |
   |---|---|
   | Entra admin center home | `https://entra.microsoft.com` |
   | All users | `https://entra.microsoft.com/#view/Microsoft_AAD_UsersAndTenants/UserManagementMenuBlade/~/AllUsers` |
   | Sign-in logs | `https://entra.microsoft.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/SignIns` |
   | Conditional Access | `https://entra.microsoft.com/#view/Microsoft_AAD_ConditionalAccess/ConditionalAccessBlade/~/Overview` |
   | Graph Explorer | `https://aka.ms/ge` |
   | My Apps (user view) | `https://myapps.microsoft.com` |

   > 💡 **Tip:** Direct URLs are more reliable than menu navigation for frequently used sections. When Microsoft reorganizes the admin center, direct URLs often still work even if the menu path changed.

**You'll know this worked when** your Favorites bar shows five pinned items and you have browser bookmarks for the six URLs above.

---

## The Series Navigation Reference

Use this table throughout the series. When a module says "navigate to Conditional Access," this is where it is.

| What you need | Location | Direct access |
|---|---|---|
| All users | Identity → Users → All users | Search: `all users` |
| Create a user | Identity → Users → All users → + New user | Search: `new user` |
| Sign-in logs | Identity → Users → Sign-in logs | Search: `sign-in logs` |
| Audit logs | Identity → Users → Audit logs | Search: `audit logs` |
| All groups | Identity → Groups → All groups | Search: `groups` |
| Guest users (B2B) | Identity → External Identities | Search: `external identities` |
| Admin roles | Identity → Roles & admins → All roles | Search: `roles` |
| Administrative Units | Identity → Roles & admins → Administrative units | Search: `administrative units` |
| Conditional Access | Protection → Conditional Access | Search: `conditional access` |
| Authentication methods | Protection → Authentication methods | Search: `authentication methods` |
| SSPR | Protection → Authentication methods → Password reset | Search: `password reset` |
| Identity Protection | Protection → Identity Protection | Search: `identity protection` |
| PIM | Protection → Privileged Identity Management | Search: `pim` |
| Access reviews | Governance → Access reviews | Search: `access reviews` |
| Access packages | Governance → Entitlement management → Access packages | Search: `access packages` |
| Lifecycle workflows | Governance → Lifecycle workflows | Search: `lifecycle workflows` |
| Enterprise applications | Applications → Enterprise applications | Search: `enterprise applications` |
| App registrations | Applications → App registrations | Search: `app registrations` |
| Devices | Devices → All devices | Search: `devices` |
| Tenant properties | Settings (gear icon) → Tenant properties | Search: `tenant properties` |
| Domain names | Identity → Settings → Domain names | Search: `domain names` |

---

## The Scenario Check

The Contoso admin workspace is configured. The admin center sections that will be used throughout the series are located and accessible. Sign-in logs, Conditional Access, PIM, and Enterprise applications are pinned to the Favorites bar for fast access. Direct URL bookmarks are set up in the browser. No Contoso users or policies exist yet. Every lab from Module 2.1 onward assumes you can navigate to any section in the table above without reading the steps.

---

## Check Your Understanding

- [ ] Navigate to **Conditional Access** using the left nav, then close the tab. Navigate there again using only the search bar.
- [ ] Navigate to **Sign-in logs** using only the Favorites bar (without the left nav or search).
- [ ] State the difference between `entra.microsoft.com` and `admin.microsoft.com` in one sentence.
- [ ] Without reading the table above, state where **Access packages** lives in the admin center.
- [ ] Without reading the table above, state where **Identity Protection** lives in the admin center.
- [ ] Name two tasks that can only be done in the Entra admin center, not the Microsoft 365 admin center.

---

## Common Mistakes

**Doing identity work in the Microsoft 365 admin center instead of the Entra admin center.**
The Microsoft 365 admin center shows users and basic properties, but it does not surface Conditional Access, Identity Protection, PIM, or authentication method settings. If you configure something in M365 admin center and cannot find it again, the feature likely lives in the Entra admin center instead.

**Relying only on the left nav and ignoring search.**
The left nav changes when Microsoft reorganizes the portal. Admins who only navigate by menu path get confused when items move. Admins who use search find things regardless of where the menu puts them. Build the search habit now.

**Assuming what is visible in the admin center is all that exists.**
The admin center surfaces the most commonly used settings. Many advanced settings are only accessible via Microsoft Graph API or PowerShell. When you cannot find a setting in the portal, it may be configurable only via Graph. You will use Graph Explorer starting in Module 1.4.

---

## What's Next

Module 1.4 covers how Entra ID makes decisions — the complete sign-in flow from the moment a user types their email to the moment they receive an access token. You will follow an actual sign-in through the logs, read every field in the log entry, and decode an access token at `jwt.ms`. Understanding this flow is the foundation for every policy you will build in Units 3 through 9.

---

> 📚 **Entra ID — Zero to Admin** · Unit 1: Getting Started
>
> [← Module 1.2 — Understanding Your Tenant](../1-2-understanding-your-tenant/1-2-understanding-your-tenant.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 1.4 — How Entra ID Makes Decisions →](../1-4-how-entra-id-makes-decisions/1-4-how-entra-id-makes-decisions.md)
