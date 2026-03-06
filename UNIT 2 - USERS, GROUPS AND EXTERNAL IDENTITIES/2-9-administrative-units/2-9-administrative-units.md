# Administrative Units — Delegated Admin Without Full Tenant Access
*The difference between "manage users in IT" and "manage all users in the tenant" is an Administrative Unit.*

> 🟡 **Intermediate** · Unit 2, Module 2.9

---

## What You Will Learn

- Explain what an Administrative Unit is and what problem it solves
- Create an Administrative Unit scoped to the IT department
- Add Jordan Kim to the Administrative Unit as a scoped User Administrator
- Verify that Jordan's administrative permissions are limited to users in the AU — not the whole tenant
- Describe when to use Administrative Units vs custom roles

---

## Why This Matters

*SC-300: Implement and manage user identities — Implement and manage administrative units*

Global Administrator is the most powerful role in Entra ID. Most organizations should have as few Global Administrators as possible — ideally two or three for redundancy and emergency access. But administrators need to be able to do their jobs. Jordan Kim manages IT at Contoso. Jordan needs to reset passwords and manage accounts for IT department users. That does not require Global Administrator — it requires a scoped User Administrator. Administrative Units (AUs) make that scope restriction possible.

> 🔑 **Licensing:** Administrative Units require **Entra ID P1**. Your developer tenant includes this. In production, verify your license before implementing AU-based delegation.

---

## 🏢 The Problem AUs Solve

Without Administrative Units, every Entra ID built-in role (User Administrator, Helpdesk Administrator, etc.) applies to the entire tenant. If you assign Jordan Kim the User Administrator role, Jordan can reset passwords for every user in the organization — Finance, HR, Engineering, everyone. In a 50-person company that might be acceptable. In a 5,000-person company with regional IT teams, it is not.

An Administrative Unit is a container. You put users, groups, or devices into the container. You assign a scoped role to an admin — but only for objects inside that container. Jordan Kim as a User Administrator scoped to the IT-AU can only manage users who are members of IT-AU. Everyone outside the AU is invisible to Jordan's administrative privileges.

Most people assume this is complicated to set up. It is not. It is a three-step process: create the AU, add objects to it, assign the scoped role.

---

## 📦 What You Can Put in an Administrative Unit

| Object type | Can be an AU member | Notes |
|---|---|---|
| Users | ✅ Yes | Most common use case |
| Groups | ✅ Yes | Adding a group to an AU does not add group members to the AU |
| Devices | ✅ Yes | Useful for scoping device management roles |

One important clarification: when you add a **group** to an AU, you are making the group object itself a member of the AU — not the users inside the group. A scoped User Administrator over that AU can manage the group object, but they cannot manage the individual user members of that group through the AU. To scope user management, add the users themselves to the AU.

---

## 🔐 Which Roles Can Be Scoped to an AU

Not all Entra ID roles support AU scoping. The roles that do:

| Role | What it does scoped to an AU |
|---|---|
| User Administrator | Create, update, delete users; reset passwords; manage group memberships (within AU) |
| Helpdesk Administrator | Reset passwords and update non-admin properties (within AU) |
| License Administrator | Assign and remove licenses (within AU) |
| Groups Administrator | Create and manage groups (within AU) |
| Authentication Administrator | Reset authentication methods for non-admin users (within AU) |
| Password Administrator | Reset passwords for non-admin users (within AU) |

Global Administrator cannot be scoped to an AU — it is always tenant-wide.

---

## Lab 2.9.1 — Create an Administrative Unit for the IT Department

**Tools:** Entra admin center
**Prerequisite:** Jordan Kim exists as an IT Administrator user (from Module 2.1)

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. In the left menu, navigate to **Identity** → **Roles & admins** → **Administrative units**.

3. Click **+ Add** at the top of the page.

4. The **Add administrative unit** panel opens. Fill in:
   - **Name:** `Contoso-IT-AU`
   - **Description:** `Administrative Unit for IT department users — scoped admin delegation`

5. Under **Assign Azure AD roles when creating this administrative unit**, leave this toggle off for now — you will assign the role in a separate step.

6. Click **Next: Assign members** at the bottom of the panel.

7. On the **Assign members** page, click **+ Add members**.

8. Search for **Jordan Kim** and select them. Click **Select**. Jordan Kim is added as a member of the AU — not as an admin of the AU. As a member, Jordan's user account is now inside the scope of the AU for management purposes.

9. Click **Next: Review + create**.

10. Review the summary. Click **Create**.

11. The Administrative Unit `Contoso-IT-AU` is created. You are returned to the Administrative units list.

12. Click on **Contoso-IT-AU** to open it. In the left menu, select **Members** (under **Manage**). Verify that **Jordan Kim** appears as a member.

---

## Lab 2.9.2 — Assign a Scoped Role to an Administrator

**Tools:** Entra admin center
**Prerequisite:** Lab 2.9.1 complete

You will assign the User Administrator role to your Global Administrator account, but scoped only to `Contoso-IT-AU`. In a real environment, you would assign this to a separate helpdesk account — not your Global Admin. Here you will assign it to your own account for demonstration, then verify the scope limitation.

1. Navigate to **Identity** → **Roles & admins** → **Administrative units** → **Contoso-IT-AU**.

2. In the left menu under **Manage**, select **Roles and administrators**.

3. You will see a list of roles that can be scoped to this AU. Click **User administrator**.

4. The User administrator assignments panel opens (scoped to this AU only). Click **+ Add assignments**.

5. In the search box, type the name of your Global Administrator account and select it. Click **Select** → **Next** → **Assign**.

6. Your Global Administrator account is now assigned User Administrator **scoped to Contoso-IT-AU**. This is in addition to — not instead of — the Global Administrator role. In a real scenario, this would be a separate helpdesk account with no other admin roles.

7. Navigate back to **Contoso-IT-AU** → **Roles and administrators**. You should see User Administrator with your account listed as an assignment.

**Verify scope from the AU perspective:**

8. Navigate to **Identity** → **Roles & admins** → **Administrative units** → **Contoso-IT-AU** → **Members**.

9. Click **+ Add members**. Search for another user — add **Alex Lee** to the AU as a member.

10. Now Contoso-IT-AU contains two members: Jordan Kim and Alex Lee. A scoped User Administrator over this AU can manage both Jordan and Alex — but cannot manage Morgan Chen, River Patel, or Sam Taylor, who are not in the AU.

---

## Lab 2.9.3 — Test the Scope Boundary

**Tools:** Entra admin center
**Prerequisite:** Lab 2.9.2 complete

This lab demonstrates what scope limitation looks like in practice. You will try to perform admin actions on users both inside and outside the AU.

**Actions inside the AU scope:**

1. Navigate to **Identity** → **Roles & admins** → **Administrative units** → **Contoso-IT-AU** → **Members**.

2. Click on **Jordan Kim** within the AU members list. This opens Jordan's user profile from within the AU scope.

3. Click **Reset password** in Jordan's profile. A temporary password can be generated. The action succeeds because Jordan is within the AU scope.

4. Click **Cancel** — do not actually reset Jordan's password.

**Understand the scope boundary with non-AU members:**

5. Navigate to **Identity** → **Users** → **All users**. Search for **Morgan Chen** and open their profile.

6. From a Global Administrator perspective, you can see Morgan's full profile and take any action. But a User Administrator scoped only to Contoso-IT-AU would not see Morgan in the AU members list, and any action on Morgan via that scoped role would be blocked.

7. To fully demonstrate the scope boundary, you would need a second admin account with only the scoped User Administrator role (and no other roles). Creating a second admin account is outside the scope of this module — but this is the exact scenario to set up in a production environment when delegating helpdesk access.

**View your role assignments:**

8. Navigate to **Identity** → **Roles & admins** → **My role assignments** (or select your admin account → **Assigned roles**).

9. You should see two entries for User Administrator:
   - One with scope: **Directory-level** (this would exist if you had the role tenant-wide — your account has Global Admin, which supersedes this)
   - One with scope: **Contoso-IT-AU** (the scoped assignment from Lab 2.9.2)

10. The scoped assignment shows that the User Administrator role is restricted to that AU — it does not extend to all users in the tenant.

---

## Scenario Check

Contoso now has an Administrative Unit structure. `Contoso-IT-AU` contains Jordan Kim and Alex Lee as member objects. A scoped User Administrator assignment over this AU means that a helpdesk admin limited to that role can only manage Jordan and Alex — not the HR or Finance users.

In a production Contoso setup, Jordan Kim would not be a user object inside the AU — Jordan would be the admin assigned to the AU, managing other IT department staff within it. For demonstration purposes, Jordan is both a member (so the scope limitation is visible) and represents the type of admin who would manage others within their department.

When Contoso grows and adds a regional IT team in another office, that office gets its own Administrative Unit with its own scoped helpdesk admin — without any of those accounts needing Global Administrator.

---

## Check Your Understanding

- [ ] Navigate to **Contoso-IT-AU** → **Members** — confirm Jordan Kim and Alex Lee appear, and Morgan Chen does not
- [ ] From memory: name three Entra ID roles that support Administrative Unit scoping
- [ ] State the difference between adding a group object to an AU vs adding individual users to an AU
- [ ] Explain why Global Administrator cannot be scoped to an Administrative Unit
- [ ] Navigate to **Roles & admins** → **Administrative units** — confirm Contoso-IT-AU appears in the list with its description
- [ ] State the license requirement for Administrative Units

---

## Common Mistakes

**Adding a group to an AU expecting it to scope management of the group's members.**
Adding a group object to an AU scopes management of the group object itself — not the user accounts that are members of that group. To scope user management, add the individual user accounts to the AU, not the group.

**Assigning Global Administrator to an AU.**
Global Administrator is always tenant-wide and cannot be scoped. If you need delegated admin limited to a subset of users, use one of the roles listed in the table above (User Administrator, Helpdesk Administrator, etc.) and scope it to an AU.

**Using AUs as a replacement for role separation.**
AUs scope the objects a role can manage. They do not reduce the power of the role within that scope. A User Administrator scoped to the Finance AU can still create users in the Finance AU, delete them, and reset their passwords — just not for users outside the AU. If the concern is preventing too much power within a group of users, a custom role (covered in Unit 7) is the right tool.

**Creating one AU for the entire organization.**
An AU containing all users in the tenant is functionally identical to no AU — a scoped admin over a tenant-wide AU can manage everyone. AUs only provide meaningful scope limitation when they contain a subset of users.

---

## What's Next

Unit 2 is complete. You have created all five Contoso characters, organized them into static and dynamic groups, licensed them through group membership, invited an external contractor as a B2B guest, and configured external collaboration controls. Unit 3 begins with authentication — specifically how MFA works, what methods are available, and how to configure the authentication policy that governs every sign-in in your tenant.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.8 — External Collaboration Settings](../2-8-external-collaboration-settings/2-8-external-collaboration-settings.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.1 — From Password to Token →](../../UNIT 3 - AUTHENTICATION/3-1-from-password-to-token/3-1-from-password-to-token.md)
