# User Properties and What They Drive
*Three properties you thought were cosmetic are running your entire access model*

> 🟢 **Beginner** · Unit 2, Module 2.2

---

## What You Will Learn

- Identify which user properties are functional (drive automation and access) vs which are informational only
- Set manager relationships for all Contoso users and verify the hierarchy in the portal
- Create a basic dynamic group rule and watch it respond to a department change in real time
- Remove a user's usage location and see exactly which operations fail as a result
- Update user properties via both the portal and PowerShell

---

## Why This Matters

*SC-300: Implement and manage user identities — Configure and manage user properties in Microsoft Entra ID*

When you created the five Contoso users in Module 2.1, you filled in department, job title, and usage location. Most people treat those fields as an HR form — label a user, move on. They are not. Department feeds dynamic group membership. Manager feeds approval workflows, lifecycle triggers, and delegation in Entitlement Management. Usage location is a hard dependency for licensing — without it, you cannot assign most Microsoft 365 licenses, and the error message is not always obvious.

Getting user properties right at creation prevents a class of access problems that are genuinely hard to diagnose weeks later.

---

## 📋 Two Buckets: Cosmetic vs Functional

Every field on a user object does one of two things: it stores information for display, or it actively drives other systems.

| Property | Type | What it drives |
|---|---|---|
| Display name | Cosmetic | How the user appears in email, Teams, portal |
| First name / Last name | Cosmetic | Display only |
| Job title | Cosmetic | Shows in profiles, org chart |
| Employee ID | Cosmetic | HR reference; no Entra behavior attached |
| **Department** | **Functional** | **Dynamic group membership rules** |
| **Usage location** | **Functional** | **Required to assign any Microsoft 365 license** |
| **Manager** | **Functional** | **Approval workflows, lifecycle triggers, delegation** |
| **User type** (Member/Guest) | **Functional** | **What the user can see and access by default** |
| **Account enabled** | **Functional** | **Whether the user can sign in at all** |
| **UPN / sign-in name** | **Functional** | **Authentication identity** |

Most configuration mistakes in Entra ID trace back to one of the functional properties being wrong or missing.

---

## 🏗️ Why Department Is More Than a Label

A dynamic group is a group whose members are determined by a rule — not by an admin manually adding names. The rule runs against user attributes. The most common rule is `department -eq "Finance"`.

Every time a user's department attribute changes, Entra ID re-evaluates every dynamic group rule that references department for that user. This happens automatically, usually within 5 minutes. No action required from an admin.

This means department is not informational. It is the membership key for every attribute-based group you build. If you spell it inconsistently ("Finance" vs "finance" vs "Finance Dept"), your dynamic groups will silently exclude the right people.

> ⚠️ **Warning:** Dynamic group rules are case-insensitive for `-eq` comparisons, but the department values shown in the portal are case-sensitive in storage. Consistent naming prevents confusion when building multi-condition rules later.

---

## 🔗 Why Manager Is Not Optional

Manager does four things in Entra ID that no other property does:

1. **Org chart** — the Entra ID and Microsoft 365 org chart is drawn directly from manager relationships. Teams and Outlook use this.
2. **Lifecycle Workflows** — when you automate onboarding and offboarding in Module 8, the workflow can automatically notify or assign tasks to the manager.
3. **Entitlement Management** — access packages in Module 8 can be configured so that a user's manager receives access requests for approval.
4. **My Staff** — a delegated admin portal that lets managers reset passwords and manage basic properties for their direct reports, without giving them a full admin role.

A user with no manager set is disconnected from all of these. In practice, this means approval requests go nowhere, org chart gaps appear in Teams, and delegated admin scenarios break.

---

## 📍 Why Usage Location Blocks Licensing

Microsoft's service agreements require that licensed services comply with local data residency and export regulations. Entra ID enforces this at the user level through usage location.

When you try to assign a Microsoft 365 license to a user with no usage location set, the assignment fails. The error in the portal is: *"You must select a usage location for this user before you assign a license."* The error in PowerShell is less obvious — the command runs without error but the license is not actually assigned.

> 🔑 **Key concept:** Usage location does not change where data is stored. It tells Microsoft which regulatory framework applies for that user's license compliance. It is a mandatory field — not a preference.

---

## Lab 2.2.1 — Set Manager Relationships for All Contoso Users

**Tools:** Entra admin center
**Prerequisite:** All five Contoso users created in Module 2.1 (Alex Lee, Morgan Chen, Jordan Kim, Sam Taylor, River Patel)

The Contoso reporting structure:
- **River Patel** (HR Manager) has no manager — report to the organization head
- **Jordan Kim** (IT Administrator) has no manager — department head
- **Alex Lee** reports to River Patel
- **Morgan Chen** reports to River Patel
- **Sam Taylor** (Contractor) reports to Jordan Kim for IT access purposes

Set Alex Lee's manager first.

1. Open the Entra admin center at `entra.microsoft.com` and sign in as your Global Administrator.

2. In the left menu, select **Identity** → **Users** → **All users**.

3. In the user list, click **Alex Lee** to open the user's profile page.

4. In the left menu of the user profile, select **Properties**. The Properties page opens showing all editable fields.

5. Scroll down to the **Job info** section. Locate the **Manager** field. It currently shows *No manager set*.

6. Click the edit icon (pencil) next to the **Manager** field. A search box opens.

7. Type **River** and select **River Patel** from the results. Click **Save**.

8. You should see a confirmation notification: *"User successfully updated."* The **Manager** field now shows River Patel's name as a clickable link.

Now set Morgan Chen's manager.

9. Navigate back to **Identity** → **Users** → **All users**.

10. Click **Morgan Chen**.

11. Select **Properties** from the left menu. Scroll to **Job info**. Click the edit icon next to **Manager**.

12. Type **River** and select **River Patel**. Click **Save**.

Now set Sam Taylor's manager.

13. Navigate to **All users** → **Sam Taylor** → **Properties** → **Job info** → **Manager**.

14. Type **Jordan** and select **Jordan Kim**. Click **Save**.

**Verify the hierarchy:**

15. Navigate to **All users** → **River Patel** → **Direct reports**.

16. You should see **Alex Lee** and **Morgan Chen** listed as River's direct reports. If they don't appear immediately, wait 1–2 minutes and refresh.

17. Navigate to **All users** → **Jordan Kim** → **Direct reports**. Verify that **Sam Taylor** appears.

---

## Lab 2.2.2 — Watch Department Drive Group Membership in Real Time

**Tools:** Entra admin center
**Prerequisite:** Lab 2.2.1 complete. All five Contoso users exist with correct departments from Module 2.1.

This lab requires a dynamic group. You will create a simple one here. Module 2.4 covers dynamic groups in full depth — this is an early preview to make the department-to-group connection concrete.

**Step 1: Create a dynamic group for Finance users**

1. In the Entra admin center, select **Identity** → **Groups** → **All groups**.

2. Click **New group** at the top of the page.

3. Fill in the fields:
   - **Group type:** Select **Security**
   - **Group name:** Type `Finance-Dynamic`
   - **Group description:** Type `All users in the Finance department — dynamic rule`
   - **Membership type:** Select **Dynamic User** (this is the key step — not Assigned)

4. Under **Dynamic user members**, click **Add dynamic query**.

5. The dynamic membership rules editor opens. Set the rule:
   - **Property:** `department`
   - **Operator:** `Equals`
   - **Value:** `Finance`

6. Click **Save** on the rule editor, then click **Create** on the group creation page.

7. The group is created. You are returned to the group list. Click **Finance-Dynamic** to open it.

8. Select **Members** from the left menu. Wait up to 5 minutes for the initial rule evaluation to complete. Refresh the page. You should see **Alex Lee** (department: Finance) listed as a member. Morgan Chen, Jordan Kim, Sam Taylor, and River Patel should not appear.

> ⚠️ If Alex Lee does not appear after 5 minutes, go to **All users** → **Alex Lee** → **Properties** and confirm the Department field shows exactly `Finance` (capital F, no trailing spaces). The rule value must match the stored attribute exactly.

**Step 2: Change Alex's department and watch the group respond**

9. Navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Properties**.

10. In the **Job info** section, find the **Department** field. Click the edit icon. Change the value from `Finance` to `Engineering`. Click **Save**.

11. Navigate back to **Identity** → **Groups** → **All groups** → **Finance-Dynamic** → **Members**.

12. Refresh the page every 2–3 minutes. Within 5 minutes, Alex Lee should disappear from the member list. The group rule re-evaluated automatically when the department attribute changed.

**Step 3: Restore Alex to Finance**

13. Navigate back to **Alex Lee** → **Properties** → **Job info** → **Department**. Change it back to `Finance`. Click **Save**.

14. Return to **Finance-Dynamic** → **Members** and verify Alex Lee reappears within 5 minutes.

> 💡 **Tip:** In a production tenant with many users and complex rules, dynamic group processing can take up to 24 hours. In your developer tenant with 5–20 users, it is typically under 5 minutes. Do not count on sub-minute propagation in production.

---

## Lab 2.2.3 — Break and Fix a Licensing Block Caused by Missing Usage Location

**Tools:** Entra admin center, PowerShell
**Prerequisite:** All five Contoso users have Usage location set to United States (from Module 2.1)

**Step 1: Remove Alex Lee's usage location**

1. Navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Properties**.

2. In the **Settings** section, find the **Usage location** field. It currently shows **United States**.

3. Click the edit icon. Change the **Usage location** dropdown to the blank/empty option at the top of the list. Click **Save**.

**Step 2: Attempt to assign a license and observe the failure**

4. In Alex's profile left menu, select **Licenses**.

5. Click **Assignments** at the top if shown, then click **+ Assignments**.

6. In the license assignment panel, check the box next to **Microsoft 365 E5 Developer** (or whichever license appears in your tenant). Click **Save**.

7. You will see an error banner: *"1 license assignment errors. 1 user was not assigned a license because they don't have a usage location set."* The license was not assigned despite the button appearing to succeed. The user now has no license.

> 🔑 **Key concept:** This is a silent failure pattern. The portal shows an error, but if you navigate away quickly or do this via a bulk operation, you might not see it. Always verify license assignment via the **Licenses** tab after assigning.

**Step 3: Fix it**

8. Navigate back to **Alex Lee** → **Properties** → **Settings** → **Usage location**. Set it back to **United States**. Click **Save**.

9. Navigate to **Licenses** → **+ Assignments**. Assign **Microsoft 365 E5 Developer** again. Click **Save**. The assignment succeeds without error.

**Step 4: Verify via PowerShell**

10. Open PowerShell and connect:
```powershell
Connect-MgGraph -Scopes "User.Read.All"
```

11. Run:
```powershell
Get-MgUser -UserId "alex.lee@yourdomain.onmicrosoft.com" -Property "DisplayName,UsageLocation,AssignedLicenses" | Select-Object DisplayName, UsageLocation, AssignedLicenses
```

Replace `yourdomain` with your actual tenant domain.

Expected output (shortened):
```
DisplayName  UsageLocation  AssignedLicenses
-----------  -------------  ----------------
Alex Lee     US             {@{DisabledPlans=System.Object[]; SkuId=...}}
```

If `AssignedLicenses` shows `{}` (empty array), the license is not applied. `US` in the `UsageLocation` column confirms the location is set correctly. If the assignment failed, re-check the portal step above.

---

## Scenario Check

All five Contoso users now have accurate manager relationships. River Patel shows Alex Lee and Morgan Chen as direct reports in both the Entra admin center and the Microsoft 365 org chart visible in Teams. Jordan Kim shows Sam Taylor as a direct report. This structure is now available for approval workflows in Module 8 — when Alex requests an access package, the request can be routed to River automatically, without any additional configuration.

The Finance-Dynamic group now contains Alex Lee and no one else. When Alex's department temporarily changed to Engineering, the group removed them within 5 minutes. When it changed back, Alex rejoined. This is the exact mechanism that will power license group assignment in Module 2.5 — no manual maintenance required as Contoso grows.

The usage location experiment confirmed that licensing is blocked silently when usage location is missing. All five users currently have Usage location set to United States. Any user created without this field set cannot be licensed for Microsoft 365 until it is added.

---

## Check Your Understanding

- [ ] Open Jordan Kim's profile and navigate to **Direct reports** — confirm Sam Taylor appears without using search
- [ ] From memory: name the three things the Manager field drives in Entra ID beyond the org chart
- [ ] Find the Finance-Dynamic group and confirm Alex Lee is the only current member
- [ ] Remove River Patel's usage location. Attempt to assign a license. Confirm the error. Restore usage location and re-assign. — Do this from memory, without referencing the lab steps above
- [ ] State the difference between **Dynamic User** and **Assigned** membership types in one sentence
- [ ] Explain why department casing consistency matters when writing dynamic group rules

---

## Common Mistakes

**Setting manager on the wrong user object.**
When two users have similar names, it is easy to set the manager on the wrong account. After setting manager, always verify by navigating to the manager's **Direct reports** page and confirming the expected name appears. Do not trust the field on the employee's profile alone — the direct reports view catches the error immediately.

**Using an inconsistent department value across users.**
If some users have `Finance` and others have `finance` or `Finance Department`, your dynamic group rules will silently exclude the wrong users. `department -eq "Finance"` does not match `finance`. Decide on exact department strings before bulk-creating users and enforce them consistently. A PowerShell check across all users is faster than finding one missing member in a group three weeks later.

**Assuming usage location is cosmetic.**
It is required for licensing. "I'll add it later" becomes "I don't know why Alex can't access Teams" two days later when someone forgets. Set usage location at user creation — every time.

**Setting manager to yourself or a shared mailbox.**
The Manager field expects a user object, not a distribution group or service account. If you point manager to an unmanned account, approval workflows in Entitlement Management will wait indefinitely for an approval that will never come.

---

## What's Next

Module 2.3 covers Security Groups vs Microsoft 365 Groups — the two group types that will carry most of what you build in this series. The Finance-Dynamic group you created here is a Security Group. Module 2.3 explains when to use Security Groups vs M365 Groups, and sets up the group structure that Module 2.4 and 2.5 build on.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.1 — Creating and Managing Users](../2-1-creating-managing-users/2-1-creating-managing-users.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.3 — Security Groups vs Microsoft 365 Groups →](../2-3-security-groups-vs-m365-groups/2-3-security-groups-vs-m365-groups.md)
