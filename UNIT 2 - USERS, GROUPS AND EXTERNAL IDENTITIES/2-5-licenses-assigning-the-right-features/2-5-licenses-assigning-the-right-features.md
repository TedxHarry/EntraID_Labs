# Licenses — Assigning the Right Features
*Group-based licensing means a new hire gets access the moment their attribute is set — not the moment an admin remembers*

> 🟢 **Beginner** · Unit 2, Module 2.5 · ⏱️ 45 minutes

---

## What You Will Learn

- Explain the difference between group-based and direct (per-user) license assignment
- Assign Microsoft 365 E5 licenses to all five Contoso users through a license group
- Watch what happens to a user's app access when they are removed from the license group
- Verify license assignment status and spot assignment errors in the portal
- Check which Entra ID features require P1 vs P2 without looking at a reference sheet

---

## Why This Matters

*SC-300: Implement and manage user identities — Manage licenses in Microsoft Entra ID*

License assignment is how you control which Microsoft 365 features a user can access. No license means no Teams, no Exchange Online, no SharePoint. Wrong license means features appear in the UI but throw errors when used. Group-based licensing is the only assignment method that scales — it ties license assignment to group membership, so onboarding and offboarding handle licensing automatically. Manually assigning licenses to individual users works in a 5-person test tenant and breaks in a 50-person real company.

---

## 📦 What a License Actually Is

A Microsoft 365 license is a bundle of service plans. When you assign an **Microsoft 365 E5** license to a user, you are enabling a collection of individual services: Exchange Online Plan 2, SharePoint Online Plan 2, Teams, Entra ID P2, Intune, Defender for Endpoint, and more.

Each service plan inside the license can be enabled or disabled individually. You might assign E5 to all users but disable the Defender for Endpoint plan for non-IT users who don't need it.

In Entra ID, you can see this in any user's **Licenses** section — the assigned license shows, and expanding it reveals the individual service plans and their enabled/disabled state.

---

## 🔗 Group-Based vs Direct Assignment

| Method | How it works | When to use | When not to use |
|---|---|---|---|
| **Direct (per-user)** | Admin manually assigns license to each individual user | Testing a single user, emergency one-off | Any environment with more than 5 users |
| **Group-based** | License is assigned to a group; all members inherit it | Standard practice in any real environment | Never inappropriate — always preferred |

Direct assignment creates a management problem. When a user offboards, someone has to remember to remove the license. When a user transfers departments, someone has to remember to change their license. Group membership changes handle both automatically.

> 🔑 **Key concept:** A user can have both a direct assignment and a group-based assignment for the same license. Entra ID does not error — it shows both. But if you remove the group assignment thinking that removes the license, the direct assignment keeps it active. Always check both sources when debugging license issues.

---

## ⚠️ License Assignment Prerequisites

Before a license can be assigned to a user — directly or through a group — three conditions must be met:

1. **Usage location is set.** A user with no usage location cannot receive a license. The assignment will fail silently in some PowerShell paths, and show an error in the portal.

2. **The tenant has available license seats.** Your developer tenant has a finite number of E5 seats. If you have 25 users and 25 licenses, the 26th user will get an assignment error.

3. **The user account is enabled.** A disabled user can technically hold a license but cannot use any of the services. This wastes a seat.

License assignment errors appear on the group's **Licenses** page and on the individual user's **Licenses** page. Always check both after assigning.

---

## 🎯 Which Features Need Which License

The features you will build in this series have specific license requirements. You need to know these without looking them up.

| Feature | Minimum license |
|---|---|
| Entra ID core (users, groups, basic SSO) | Free |
| Conditional Access | Entra ID P1 |
| Risk-based Conditional Access | Entra ID P2 |
| Identity Protection (risky users, sign-in risk) | Entra ID P2 |
| Privileged Identity Management (PIM) | Entra ID P2 |
| Access Reviews | Entra ID P2 |
| Entitlement Management and Access Packages | Entra ID P2 |
| Lifecycle Workflows | Entra ID P2 |
| Administrative Units | Entra ID P1 |
| App provisioning (SCIM) | Entra ID P1 |
| Group-based licensing | Entra ID P1 |

Your developer tenant includes Entra ID P2 (bundled in E5), so all features are available for every lab. In a production environment, check license tier before building on any P1 or P2 feature.

---

## Lab 2.5.1 — Assign Licenses via Group

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** Finance-Dynamic group exists with Alex Lee as a member. All five Contoso users have Usage location set to United States.

> 🔑 **Licensing:** Group-based license assignment requires **Entra ID P1**. Your developer tenant includes this.

You will assign the Microsoft 365 E5 license to a group that covers all Contoso users. Since your developer tenant likely has only five users, a single all-users group is appropriate.

**Step 1: Create an all-users license group**

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Identity** → **Groups** → **All groups** → **New group**.

3. Configure:
   - **Group type:** Security
   - **Group name:** `Contoso-Licensed`
   - **Group description:** `All licensed Contoso users — Microsoft 365 E5`
   - **Membership type:** Assigned

4. Click **No members selected**. Add all five Contoso users: Alex Lee, Morgan Chen, Jordan Kim, Sam Taylor, River Patel. Click **Select**.

5. Click **Create**.

**Step 2: Assign the license to the group**

6. Navigate to **Identity** → **Groups** → **All groups** → **Contoso-Licensed**.

7. In the left menu, select **Licenses**.

8. Click **Assignments** at the top, then click **+ Assignments**.

9. A panel opens showing available licenses in your tenant. Check the box next to **Microsoft 365 E5 Developer** (or the license name shown in your tenant — it may appear as **Microsoft 365 E5** or **Office 365 E5**). Click **Save**.

10. The license is now assigned to the group. Entra ID begins processing the assignment for each member. This can take 5–10 minutes.

**Step 3: Verify the assignment for each user**

11. Navigate to **Identity** → **Users** → **All users** → **Alex Lee**.

12. In the left menu, select **Licenses**. After processing completes, you should see **Microsoft 365 E5 Developer** listed with a status of **Active**.

13. Click the license name to expand it. You will see a list of individual service plans (Exchange Online, Teams, SharePoint, Entra ID Premium P2, etc.) with each one showing **On** or **Off**.

14. Repeat steps 11–13 for Morgan Chen, Jordan Kim, Sam Taylor, and River Patel. All five should show the license as Active.

**Step 4: Check for assignment errors on the group**

15. Navigate back to **Contoso-Licensed** → **Licenses** → **Assignments**.

16. You will see a summary: number of users with the license applied, and any errors. If the error count is greater than zero, click on the error count to see which users had issues and why.

17. The most common error shown here is missing usage location. If any user shows an error, navigate to their profile, set Usage location to **United States**, and the license assignment will automatically retry within a few minutes.

---

## Lab 2.5.2 — Remove a User from the License Group and Observe

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 2.5.1 complete. All five users have active licenses.

This lab shows what happens to a user's access when their license is removed.

1. Navigate to **Identity** → **Groups** → **All groups** → **Contoso-Licensed** → **Members**.

2. Find **Alex Lee** in the member list. Click the checkbox next to their name to select them.

3. Click **Remove members** at the top. Confirm the removal. Alex Lee is removed from the group.

4. Navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Licenses**.

5. The license status changes from **Active** to showing no active licenses. This happens within a few minutes. If it still shows Active immediately after removal, wait 2 minutes and refresh.

6. Scroll down on Alex's Licenses page. You will see a message indicating no licenses are assigned. If Alex tried to open Microsoft Teams or SharePoint at this point, they would see an error or degraded experience.

**Restore the license:**

7. Navigate to **Contoso-Licensed** → **Members** → **Add members**. Search for **Alex Lee** and add them back.

8. Navigate to **Alex Lee** → **Licenses**. Within 5 minutes, the Microsoft 365 E5 license should return to **Active** status. This is the full group-based license lifecycle: join group → get license; leave group → lose license.

> ⚠️ **Warning:** There is a 30-day grace period after a license is removed. During that period, the user's data (email, OneDrive files) is preserved. After 30 days, Microsoft begins permanent deletion. In a production offboarding workflow, license removal should be part of the account disable process — not delayed.

---

## Scenario Check

All five Contoso users now have Microsoft 365 E5 licenses assigned through the Contoso-Licensed group. Alex Lee, Morgan Chen, Jordan Kim, Sam Taylor, and River Patel can access Teams, SharePoint, Exchange, and all P2 Entra ID features used throughout this series.

The license assignment is group-driven. When Contoso hires a sixth employee and an admin adds them to Contoso-Licensed, they receive the license automatically — no separate license assignment step. When an employee leaves, removing them from Contoso-Licensed revokes their license.

Alex Lee's temporary removal from the license group demonstrated the offboarding effect. The license disappeared from their profile within minutes of leaving the group, and was restored just as quickly when they were re-added. This pattern will be formalized in Module 8's lifecycle workflows.

---

## Check Your Understanding

- [ ] Navigate to **Contoso-Licensed** → **Licenses** — confirm zero assignment errors without referencing the lab steps
- [ ] From memory: state three conditions that must be met before a license assignment will succeed
- [ ] Open Alex Lee's Licenses page and find the individual service plan list — identify which plan provides Entra ID Premium P2 access
- [ ] Name two differences between direct license assignment and group-based license assignment
- [ ] Explain what happens to a user's data if their license is removed and not restored within 30 days
- [ ] From memory: which Entra ID feature requires P2 but is NOT included in a P1 license?

---

## Common Mistakes

**Assigning licenses directly to users after setting up group-based assignment.**
Once you have group-based licensing, any direct assignments create a duplicate. The user appears to have two assignments — the direct one and the group one. When you remove the group in a future cleanup, the direct assignment keeps the license active and nobody can figure out why the user still has access.

**Creating a license group with Assigned membership and forgetting to update it.**
If Contoso-Licensed is Assigned (not Dynamic), every new hire must be manually added. The better long-term approach: a dynamic group that includes all member user accounts automatically (`user.userType -eq "Member"`), assigned the license. Then every internal user gets the license the moment their account is created.

**Ignoring license assignment errors on the group.**
The group's Licenses page shows how many users had errors. Most admins assign the license, see no immediate popup error, and assume everything worked. Check the assignment status page after every group license assignment — silently failed assignments are common when usage locations are inconsistent.

**Disabling service plans without documenting why.**
E5 licenses include services your organization may not use. Disabling them conserves access control and prevents accidental use of unlicensed features. But if you disable a plan and someone later re-enables it at the user level, the group-level setting is overridden. Document every service plan customization.

---

## What's Next

Module 2.6 covers bulk operations — creating, updating, and disabling multiple users at once using CSV uploads and PowerShell scripts. You will create 10 test users in a single operation, bulk-update their properties, and delete and restore them. The licensing infrastructure from this module will then be used to apply licenses to the bulk-created users.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.4 — Dynamic Groups](../2-4-dynamic-groups/2-4-dynamic-groups.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.6 — Bulk Operations →](../2-6-bulk-operations/2-6-bulk-operations.md)
