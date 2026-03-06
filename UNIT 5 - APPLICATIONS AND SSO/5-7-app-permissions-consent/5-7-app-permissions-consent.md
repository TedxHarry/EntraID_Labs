# App Permissions and Admin Consent
*Apps don't get permissions automatically. You decide what each app can do and what data it can access.*

> 🟡 **Intermediate** · Unit 5, Module 5.7 · ⏱️ 50 minutes

---

## What You Will Learn

- Distinguish between delegated permissions (app acts on behalf of user) and application permissions (app acts as itself)
- Understand the difference between user consent and admin consent
- Audit what permissions apps have been granted in your tenant
- Identify over-permissioned apps and understand the security risk
- Revoke permissions from apps

---

## Why This Matters

*SC-300: Plan and implement access management for applications — Manage app permissions and consent flow*

An app with too much permission is a security risk. If a compromised app has `User.ReadWrite.All` permission (read and modify all user accounts), an attacker can modify every user in your tenant — change passwords, add themselves as admin, grant themselves global access. Auditing app permissions is an essential security task. This module teaches you to identify and remediate over-permissioned apps.

---

## 📋 Two Types of Permissions

**Delegated Permissions** — The app acts *on behalf of the signed-in user*

- Example: "Read my email" (the app reads only the signed-in user's mailbox)
- Granted by: The user (or admin if user consent is disabled)
- Scope: User's own data only
- Lifetime: Only while the app is being used
- Examples: `Mail.Read`, `Calendar.Read`, `User.Read`

**Application Permissions** — The app acts *as itself*, not on behalf of a user

- Example: "Read all users in the tenant" (the app reads all user accounts, not just the signed-in user)
- Granted by: Admin only (always requires admin consent)
- Scope: All tenant data
- Lifetime: Permanent (until revoked)
- Examples: `User.ReadWrite.All`, `Mail.ReadWrite.All`, `Directory.ReadWrite.All`

**Rule:** Delegated permissions are safer (user scope). Application permissions are more powerful (tenant scope) and require admin oversight.

---

## 🔐 Admin Consent vs User Consent

**User Consent:**
- A user signs in to an app for the first time
- The app asks: "Can I read your email?"
- The user clicks "Allow" or "Deny"
- Only that user grants permission
- The app can act only for that user

**Admin Consent:**
- An admin explicitly grants permissions to an app
- Applies to all users in the tenant
- Required for high-risk permissions (User.ReadWrite.All, Directory.ReadWrite.All)
- Users cannot override — if admin said no, users cannot grant permission

**Why admin consent for sensitive permissions?** If users could grant `User.ReadWrite.All` to an app, a malicious app could trick users into granting it "read email" while secretly asking for "modify all users."

---

## ⚠️ What Over-Permissioning Looks Like

**Bad example:**
- Slack (an internal chat app) has permission: `User.ReadWrite.All` (read and modify all user accounts)
- If Slack is compromised, an attacker can modify every user in Contoso
- Slack doesn't need this permission — it only needs `User.Read` (read basic profile info)

**Good example:**
- HR system has permission: `User.Read.All` (read all user attributes)
- HR system has permission: `User.ReadWrite.All` (modify user attributes)
- Why? HR needs to update employee information (phone, manager, department)
- This is necessary and appropriate for HR

**Guideline:** Every app should have *exactly* the permissions it needs — no more. This is called the **principle of least privilege**.

---

## Lab 5.7.1 — Audit App Permissions in Your Tenant

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 5.1 complete; you have apps configured (Slack, etc.)

1. Navigate to **Applications** → **Enterprise applications**.

2. Click **Permissions** in the left menu (or search for "Permissions" at the top).

3. You'll see a list of apps and permissions they've been granted. This is your permission audit.

4. Review the apps listed:
   - **Slack** — Note the permissions granted (hopefully just delegated permissions like `User.Read`)
   - **Contoso Intranet Portal** — Note permissions from Module 5.5
   - **Microsoft services** (Teams, SharePoint, Exchange) — Typically have broad permissions (necessary for Microsoft services)

5. For each app, note:
   - **Permission type** — Delegated or Application
   - **Permission level** — `User.Read` (safe) vs `User.ReadWrite.All` (powerful)
   - **Consent type** — User consent vs Admin consent
   - **Consented by** — Date admin granted it

6. Identify any app with `User.ReadWrite.All` or `Directory.ReadWrite.All`:
   - Is this app supposed to modify user accounts? (HR, identity sync services)
   - If not, it's over-permissioned and should be revoked

---

## Lab 5.7.2 — Review Permissions on a Specific App

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.7.1 complete

1. Navigate to **Applications** → **Enterprise applications** → **Slack**.

2. On the left menu, click **Permissions** (or **API permissions** depending on your portal version).

3. You'll see a list of permissions Slack has been granted. For example:
   - `email` (delegated)
   - `profile` (delegated)
   - `User.Read` (delegated)

4. For each permission, note:
   - **Type** — Delegated or Application
   - **Consent status** — Consented or not consented
   - **When consented** — Date/time

5. **Question:** Does Slack need all these permissions, or are some unnecessary?
   - Slack needs user email and profile to sign in (delegated) ✓
   - Slack should NOT need to modify users or read all users (no Application permissions) ✓

6. If Slack had a permission like `Directory.ReadWrite.All`, it would be over-permissioned and should be removed.

---

## Lab 5.7.3 — Understand Permission Risk with PowerShell

**Time:** 15 minutes
**Tools:** PowerShell, Microsoft Graph PowerShell SDK
**Prerequisite:** Module 2.1 complete (PowerShell installed and connected)

You'll list all apps in your tenant and identify which ones have high-risk permissions.

1. Open **PowerShell** as Administrator.

2. Connect to Microsoft Graph:
   ```powershell
   Connect-MgGraph -Scopes "Application.Read.All"
   ```

3. List all service principals (apps) and their permissions:
   ```powershell
   Get-MgServicePrincipal -All | Select-Object DisplayName, AppId, @{
       Name = "HighRiskPermissions"
       Expression = {
           (Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $_.Id) |
           Where-Object { $_.AppRoleId -match "User.ReadWrite.All|Directory.ReadWrite.All|Mail.ReadWrite.All" } |
           Select-Object -ExpandProperty Principal
       }
   }
   ```

4. Review the output. Look for apps with high-risk permissions (anything with `ReadWrite.All` or `Directory`).

5. For each app with high-risk permissions, ask: "Does this app legitimately need this permission?"
   - Entra Connect Sync → Yes, it syncs user accounts
   - HR Information System → Yes, it updates employee data
   - Slack → No, it should never have `User.ReadWrite.All`

6. If an app is over-permissioned, note it for revocation (next lab).

---

## Lab 5.7.4 — Revoke Permissions from an App

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.7.3 complete; you've identified an over-permissioned app

If you discovered an app with unnecessary permissions, here's how to remove them.

**Scenario:** You find that a test app accidentally has `User.ReadWrite.All`. You want to revoke it.

1. Navigate to **Applications** → **Enterprise applications** → search for the test app.

2. Click on the app.

3. On the left menu, click **Permissions**.

4. Find the permission you want to revoke (e.g., `User.ReadWrite.All`).

5. Click the **X** or **Remove** button next to it.

6. Confirm the removal.

7. The permission is now revoked. The app no longer has it.

> ⚠️ **Warning:** Revoking permissions can break the app if it relies on them. Always verify the app has an alternative way to function before revoking. For non-essential permissions, revoke freely. For core permissions, test the app after revocation.

---

## The Scenario Check

Contoso has audited its app permissions. Slack has delegated permissions only (`email`, `profile`, `User.Read`) — appropriate for a chat tool. The Contoso Intranet Portal has `openid` and `profile` — appropriate for an internal app. Microsoft services (Teams, SharePoint) have broader permissions, which is necessary.

Jordan Kim (IT admin) discovered that an older app called "Travel Expense Automation" had `Directory.ReadWrite.All` permission — far more than needed for expense reporting. Jordan revoked it. The app still functions (it only needed to read employee names for approval routing), and Contoso's security posture improved.

When a new SaaS vendor asks for permission to "integrate with Microsoft 365," Contoso now knows to ask specifically what permissions they need and to audit them periodically.

---

## Check Your Understanding

- [ ] Navigate to **Permissions** and find an app with delegated permissions and an app with application permissions
- [ ] Explain the difference between delegated and application permissions in one sentence
- [ ] Identify one app in your tenant that might be over-permissioned and explain why
- [ ] State who grants admin consent and who grants user consent
- [ ] Describe the security risk of an app with `Directory.ReadWrite.All` permission

---

## Common Mistakes

**Granting `User.ReadWrite.All` when only `User.Read` is needed.**
A vendor says "We need to read and modify user attributes." You grant `User.ReadWrite.All`. Later you realize they only needed to *read* names and email addresses for reports. You gave them write access unnecessarily. Always ask "Do you need to *modify* user data, or just read it?"

**Not revoking permissions from decommissioned apps.**
You replace an HR system with a new one. The old HR system still has `User.ReadWrite.All` permission. No one uses it anymore, but it still has access. If someone gains access to the old app's credentials, they can modify all users. Always revoke permissions from apps you're retiring.

**Assuming Microsoft service apps are over-permissioned.**
Teams, SharePoint, and Exchange have broad permissions because they are core Microsoft services. Don't assume they're over-permissioned — they're designed to have those permissions. Focus audit effort on third-party apps, not Microsoft services.

**Not testing app functionality after revoking permissions.**
You revoke `User.Read.All` from an app thinking it didn't need it. The app's report-building feature breaks. Test apps after revoking permissions, don't just revoke and hope.

---

## What's Next

Module 5.8 covers **Automated Provisioning with SCIM** — how to automatically create and update user accounts in SaaS apps. Instead of manually adding users to Slack one by one, you configure Slack to receive a feed of users from Entra ID and keeps in sync. This is the enterprise way to manage app users at scale.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.6 — User Assignment](../5-6-user-assignment/5-6-user-assignment.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.8 — Automated Provisioning with SCIM →](../5-8-provisioning-scim/5-8-provisioning-scim.md)
