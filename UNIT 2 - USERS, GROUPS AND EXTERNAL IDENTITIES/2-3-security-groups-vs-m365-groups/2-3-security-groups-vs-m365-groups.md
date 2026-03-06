# Security Groups vs Microsoft 365 Groups
*Choosing the wrong group type is the most common configuration mistake in Entra ID*

> 🟢 **Beginner** · Unit 2, Module 2.3

---

## What You Will Learn

- State the exact purpose of Security Groups vs Microsoft 365 Groups and when to use each
- Create a Security Group for Finance users and an M365 Group for the Engineering team
- Add Contoso users to each group with the correct membership type
- Assign SharePoint site access via group membership rather than direct user assignment
- Explain why M365 Groups are the wrong choice for Conditional Access and license assignment

---

## Why This Matters

*SC-300: Implement and manage user identities — Create and manage groups in Microsoft Entra ID*

Entra ID has two group types and every experienced admin has a story about the wrong one being used in production. Security Groups drive access — Conditional Access policy targets, license assignments, app role assignments, SharePoint permissions. Microsoft 365 Groups drive collaboration — they create a Teams workspace, a shared mailbox, a SharePoint site, a shared calendar, and a Planner board simultaneously. Using an M365 Group where a Security Group belongs adds a Teams channel, a mailbox, and a SharePoint site you didn't ask for. Using a Security Group where an M365 Group belongs means users have no shared workspace.

---

## 🔒 What Security Groups Actually Do

A Security Group is a container for identities. You put users, devices, service principals, or other groups into it, and then you point policies and permissions at the group instead of at individuals.

Every time you add a user to a Security Group, that user inherits whatever the group grants. Every time you remove them, they lose it. This is the foundation of group-based access control.

Security Groups are the correct choice for:
- Targeting Conditional Access policies
- Assigning Microsoft 365 licenses
- Granting roles in SharePoint, Teams, or any app
- Dynamic membership rules based on attributes
- Nested group structures

Security Groups have no collaboration features. No shared mailbox. No Teams channel. No calendar.

---

## 🤝 What Microsoft 365 Groups Actually Create

When you create a Microsoft 365 Group, Entra ID creates five things at once:

| What gets created | What it's for |
|---|---|
| The group object in Entra ID | Identity membership and policy targeting |
| A shared mailbox | Receive and send email to the group address |
| A SharePoint team site | Shared document storage |
| A Teams workspace | Real-time chat and meetings (if Teams is provisioned) |
| A Planner board | Task management |

This is the right choice when a team of people needs to work together. It is the wrong choice when you want to assign a license or target a Conditional Access policy, because the extra collaboration infrastructure is irrelevant and will generate Teams channels and mailboxes nobody asked for.

> ⚠️ **Warning:** Microsoft 365 Groups cannot be used as dynamic user groups with the `-eq` operator on user attributes. If you need auto-membership based on department, you need a Security Group with dynamic membership type — not an M365 Group.

---

## 📊 The Comparison You Need Before Choosing

| Capability | Security Group | Microsoft 365 Group |
|---|---|---|
| Conditional Access policy target | ✅ Yes | ✅ Yes (but avoid — adds overhead) |
| License assignment (group-based) | ✅ Yes | ❌ No |
| Dynamic membership (user attributes) | ✅ Yes | ❌ No |
| SharePoint permission assignment | ✅ Yes | ✅ Yes (site is auto-created) |
| Creates a shared mailbox | ❌ No | ✅ Yes |
| Creates a Teams workspace | ❌ No | ✅ Yes (when provisioned) |
| App role assignment | ✅ Yes | ✅ Yes |
| Nested group support (group in group) | ✅ Yes | ❌ No |
| Device members | ✅ Yes | ❌ No |
| Guest members | ✅ Yes | ✅ Yes |

The one case where M365 Groups make sense for access: assigning permissions to a team's SharePoint site. The site was already created by the M365 Group — the group membership and the site permissions are the same thing.

---

## 🧩 Nested Groups and Why They Matter

A Security Group can be a member of another Security Group. This is nesting.

The Finance-Dynamic group from Module 2.2 contains Alex Lee. If you put Finance-Dynamic inside a larger group called Contoso-AllStaff, Alex automatically inherits anything Contoso-AllStaff grants. When Alex's department changes back to Engineering, they leave Finance-Dynamic, and automatically lose everything Finance-Dynamic granted — without touching Contoso-AllStaff.

This is how enterprise-scale access control works. You build small, attribute-driven groups and compose them into larger structures. You never manually add individuals to high-level groups.

> 💡 **Tip:** Microsoft 365 Groups do not support nesting. If your access model requires a group-of-groups structure, it must be Security Groups all the way down.

---

## Lab 2.3.1 — Create a Security Group for Finance

**Tools:** Entra admin center
**Prerequisite:** Alex Lee exists with Department = Finance (from Module 2.1)

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. In the left menu, select **Identity** → **Groups** → **All groups**.

3. Click **New group**.

4. Configure the group:
   - **Group type:** Select **Security**
   - **Group name:** Type `Finance-All`
   - **Group description:** Type `All Finance department users — assigned membership`
   - **Membership type:** Select **Assigned** (this is manual membership, not dynamic)
   - **Owners:** Click **No owners selected**. Search for your admin account (the one you're signed in as). Select it. Click **Select**.
   - **Members:** Click **No members selected**. Search for **Alex Lee**. Select Alex Lee. Click **Select**.

5. Click **Create**. The group is created. You are returned to the group list.

6. Click **Finance-All** to open the group.

7. Select **Members** from the left menu. Verify that **Alex Lee** appears. If not, wait 1 minute and refresh.

**Note on two Finance groups:**
You now have two Finance groups: Finance-Dynamic (dynamic, from Module 2.2) and Finance-All (assigned, from this lab). Both contain Alex Lee. In Module 2.5, you will use Finance-Dynamic for license assignment — dynamic groups update automatically when users are hired or transferred. Finance-All is here to contrast the membership types and will be used for SharePoint permission assignment in Lab 2.3.3.

---

## Lab 2.3.2 — Create a Microsoft 365 Group for Engineering

**Tools:** Entra admin center
**Prerequisite:** Morgan Chen exists with Department = Engineering (from Module 2.1)

1. In the Entra admin center, select **Identity** → **Groups** → **All groups**.

2. Click **New group**.

3. Configure the group:
   - **Group type:** Select **Microsoft 365**
   - **Group name:** Type `Engineering-Team`
   - **Group description:** Type `Engineering team collaboration workspace`
   - **Group email address:** Entra ID auto-fills based on the name. Accept the suggested address or type `engineering-team`
   - **Membership type:** The only option available for M365 Groups is **Assigned** — note that Dynamic User is greyed out
   - **Owners:** Click **No owners selected**. Add your admin account. Click **Select**.
   - **Members:** Click **No members selected**. Add **Morgan Chen**. Click **Select**.

4. Click **Create**.

5. Open **Engineering-Team** from the group list. Notice the group overview shows a **Teams** section and a **SharePoint site** link — these were provisioned automatically. A Security Group would not show these.

6. Select **Members**. Verify Morgan Chen appears.

**Observe what was created:**

7. Navigate to the **Overview** tab of Engineering-Team. You will see options for **Teams** and **SharePoint site** that do not exist on Security Groups. These were created automatically when the M365 Group was created.

> 💡 **Tip:** If you navigate to `admin.microsoft.com` → **Groups** → **Active groups**, you will see Engineering-Team there as well. M365 Groups are visible in both the Entra admin center and the Microsoft 365 admin center. Security Groups only appear in the Entra admin center.

---

## Lab 2.3.3 — Assign SharePoint Access via Group

**Tools:** Entra admin center, SharePoint admin center
**Prerequisite:** Finance-All Security Group exists with Alex Lee as a member

This lab demonstrates the correct pattern: grant access to a group, not to an individual. When Alex transfers out of Finance, you remove them from Finance-All and they lose access automatically — no manual permission cleanup.

1. Open the SharePoint admin center. Navigate to `admin.microsoft.com`, expand **Admin centers** in the left menu, and select **SharePoint**. This opens `admin.microsoft.com/sharepoint`.

2. In the SharePoint admin center, select **Sites** → **Active sites**.

3. You will see the Engineering-Team SharePoint site that was auto-created in Lab 2.3.2. Click on **Engineering-Team** to open its settings panel.

4. In the site settings panel, select **Membership** tab. You will see Engineering-Team group is already listed as a member with **Edit** permissions. This happened automatically — the M365 Group and its SharePoint site share membership.

5. This is the difference from a Security Group assignment: with M365 Groups, site permissions and group membership are the same. With a Security Group, you would grant the group access to a pre-existing site.

6. To see how Security Group access works on a SharePoint site, navigate back to **Active sites**. Find any site that is not tied to an M365 Group (a Communication Site, or create one if none exist). Select the site. In the settings panel, open **Settings** → **Permissions** → **Advanced permissions settings** (this opens the classic SharePoint permissions page).

7. Click **Grant Permissions** at the top. In the search box, type `Finance-All`. Select the **Finance-All** Security Group. Set **Share** permissions to **Read**. Uncheck **Send an email invitation**. Click **Share**.

8. Alex Lee now has read access to this SharePoint site through group membership. If you remove Alex from Finance-All, they lose access immediately — no separate permission change needed.

---

## Scenario Check

Contoso now has two active groups used for access control:

**Finance-All** (Security Group, assigned) contains Alex Lee. This group will be assigned to SharePoint sites, licensed apps, and Conditional Access scope as Contoso adds more Finance users to the tenant. The assigned membership type means an admin manually controls who is in the group — appropriate for small, precisely controlled access.

**Finance-Dynamic** (Security Group, dynamic) also contains Alex Lee and automatically updates as department attributes change. This will become the licensing group in Module 2.5.

**Engineering-Team** (Microsoft 365 Group) contains Morgan Chen and has a provisioned Teams workspace and SharePoint site. Morgan can access shared Engineering documents and has a Teams channel for collaboration. This group is appropriate for Engineering because they need a shared workspace — not just access control.

Jordan Kim, Sam Taylor, and River Patel are not yet in these groups — they will be added in their relevant modules as their access scenarios require.

---

## Check Your Understanding

- [ ] Open Finance-All and Engineering-Team. Without looking at the guide: state which one has a SharePoint site listed in its overview and which one does not
- [ ] From memory: name three things you can do with a Security Group that you cannot do with a Microsoft 365 Group
- [ ] Create a new Security Group called `IT-Admins` with Jordan Kim as the only member. Do this from memory
- [ ] Navigate to Engineering-Team and find where Microsoft displays the Teams workspace and SharePoint site links on the group overview page
- [ ] Explain in one sentence why you should not use an M365 Group for assigning Microsoft 365 licenses

---

## Common Mistakes

**Using a Microsoft 365 Group as a Conditional Access policy target.**
You can technically target a CA policy at an M365 Group, but every member of that group now has a shared mailbox, a Teams channel, and a SharePoint site you didn't intend to create. Use Security Groups for policy targeting.

**Expecting dynamic membership on an M365 Group.**
It is not supported. When you create an M365 Group, the **Dynamic User** option in the membership type dropdown is disabled. If your team needs auto-membership based on department or other attributes, create a Security Group with dynamic membership, not an M365 Group.

**Creating a new group for every permission assignment.**
In small tenants this seems manageable. In a 50-person company, this creates 40 groups with overlapping membership that nobody can audit six months later. The correct pattern: one group per logical access boundary (Finance users, Engineering team), then assign that group to multiple resources. Not one group per resource.

**Assigning licenses directly to users instead of through a group.**
Direct user assignment works for 5 people. At 50 people, an admin is manually tracking every new hire and every departure. Group-based license assignment (covered in Module 2.5) means the license follows group membership automatically.

---

## What's Next

Module 2.4 covers dynamic group rules in depth — how to write them, how to test them, how to handle complex multi-condition rules, and what happens when a rule is wrong. The Finance-Dynamic group from Module 2.2 becomes the starting point, and you will write progressively more complex rules that combine department, job title, and country attributes.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.2 — User Properties and What They Drive](../2-2-user-properties-and-what-they-drive/2-2-user-properties-and-what-they-drive.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.4 — Dynamic Groups →](../2-4-dynamic-groups/2-4-dynamic-groups.md)
