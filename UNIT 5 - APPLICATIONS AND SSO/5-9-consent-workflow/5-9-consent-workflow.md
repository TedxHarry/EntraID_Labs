# Consent Settings and Admin Consent Workflow
*Control whether users can grant apps permission to access their data, or whether apps require admin approval.*

> 🟡 **Intermediate** · Unit 5, Module 5.9

---

## What You Will Learn

- Explain the difference between user consent and admin consent
- Describe the three tenant-wide consent policies
- Configure a tenant-wide consent policy
- Set up an admin consent workflow so admins review app permission requests
- Understand the security trade-off between convenience and control

---

## Why This Matters

*SC-300: Plan and implement access management for applications — Manage consent and permission policies for applications*

By default, users can grant apps permission to access their data ("read my email," "access my calendar"). This is convenient but risky — a malicious app can trick users into granting broad permissions. This module teaches you to control whether users can grant permissions at all, and how to set up an approval workflow so admins review sensitive requests. This is the final governance layer for applications.

---

## 🔐 Three Consent Policies

Every Entra ID tenant has a **consent policy** that controls whether users can grant permissions to apps.

| Policy | What Users Can Do | When to Use |
|---|---|---|
| **Allow user consent for apps** | Users can grant any permission to any app | Small orgs, internal tools only, high trust |
| **Allow user consent only for Microsoft-owned apps** | Users can grant permissions to Microsoft apps only; third-party apps require admin approval | Most enterprises (balanced) |
| **Require admin consent for all apps** | No user can grant any permission; all apps require admin approval | High-security orgs, highly regulated industries |

**Default:** Most new tenants default to "Allow user consent for Microsoft-owned apps."

---

## 📋 How Admin Consent Workflow Works

**Scenario:** A user tries to sign in to a third-party app that requests broad permissions.

**Step 1: User attempts sign-in**
- User clicks "Sign in with Contoso"
- App requests permission: "Read all your data"

**Step 2: Entra ID checks the consent policy**
- Policy says: "Third-party apps require admin consent"
- User is not an admin
- User cannot grant this permission

**Step 3: Admin gets notified**
- Entra ID sends a notification to admins: "App XYZ requested permission"
- Admin reviews the request: "Should we allow this?"

**Step 4: Admin approves or denies**
- Admin: "Yes, approve" → App gets permission, user can sign in
- Admin: "No, deny" → App is blocked, user cannot sign in

**Step 5: User retries**
- User tries to sign in again
- If approved: Permission granted, sign-in succeeds
- If denied: User sees "Request denied by admin"

---

## Lab 5.9.1 — Configure Tenant-Wide Consent Policy

**Tools:** Entra admin center
**Prerequisite:** Module 5.1 complete

1. Navigate to **Applications** → **Enterprise applications** → **Consent and permissions** (or search for "User consent settings").

2. You'll see the current consent policy. Options are:
   - **Allow user consent for apps from Microsoft only**
   - **Allow user consent for all apps**
   - **Do not allow user consent**

3. For Contoso (an enterprise), select **Allow user consent for apps from Microsoft only**.

4. This means:
   - Users can grant permissions to Microsoft apps (Teams, Outlook, etc.)
   - Third-party apps (Slack, Salesforce, GitHub) require admin consent
   - Balances usability with security

5. Click **Save**.

---

## Lab 5.9.2 — Enable the Admin Consent Workflow

**Tools:** Entra admin center
**Prerequisite:** Lab 5.9.1 complete

The admin consent workflow sends notifications to admins when users request app permissions.

1. In the **Consent and permissions** page, find **Admin consent requests** section.

2. Look for setting: **Users can request admin consent for applications they are unable to consent to**

3. Toggle to **Yes** (enable it).

4. Under **Admins will receive requests from** (if visible), ensure admins are configured:
   - By default, this notifies all Global Administrators
   - Optionally, you can set specific admins (Application Administrator, Cloud Application Administrator roles)

5. Click **Save**.

Now when a user tries to use a third-party app and gets a "requires admin consent" error, they can request approval. The request goes to admins.

---

## Lab 5.9.3 — Test User Consent (Blocked Request)

**Tools:** InPrivate browser, a third-party app
**Prerequisite:** Lab 5.9.2 complete

You'll test what happens when a user tries to use a third-party app that's blocked by consent policy.

**Scenario:** Morgan Chen tries to sign in to GitHub using Entra ID.

1. Open an **InPrivate** browser window.

2. Go to GitHub.com. Click "Sign in with Entra ID" (or similar if available).

3. Sign in as **morgan.chen@yourdomain.onmicrosoft.com**.

4. GitHub requests permission: "Access your profile, email, and groups."

5. **Expected result:** Since the consent policy blocks third-party apps, you should see:
   - Error message: "Admin consent required" or "Your request has been sent to your administrator"
   - Option to "Request admin approval" or similar

6. If the option exists, click **Request admin approval**. A notification is sent to Contoso's admins.

7. If blocked outright, the error confirms the consent policy is working.

---

## Lab 5.9.4 — Review and Approve Consent Requests (Admin Side)

**Tools:** Entra admin center
**Prerequisite:** Lab 5.9.3 complete; at least one consent request generated

1. Sign in to the Entra admin center as a Global Administrator.

2. Navigate to **Applications** → **Enterprise applications** → **Consent and permissions** → **Admin consent requests**.

3. You should see any pending requests from users (like Morgan's request from Lab 5.9.3).

4. For each request, you see:
   - **App name** — The app requesting permission
   - **Requested by** — The user who made the request
   - **Permissions requested** — What the app wants
   - **Date** — When the request was made

5. Click on a request to see details:
   - Full permission list (what the app wants to access)
   - App publisher (who built the app)
   - Risk assessment (if available — does this look trustworthy?)

6. Decision:
   - **Approve** — Grant the permission; user can now sign in to the app
   - **Deny** — Reject the request; user cannot sign in

7. Click **Approve** for the GitHub request (assuming you trust GitHub).

8. The consent is granted. Morgan can now sign in to GitHub.

---

## Lab 5.9.5 — Audit Consent Decisions

**Tools:** Entra admin center
**Prerequisite:** Lab 5.9.4 complete

After approving/denying requests, audit what permissions have been granted.

1. Navigate to **Applications** → **Permissions** (same place you audited in Module 5.7).

2. If you approved GitHub in Lab 5.9.4, you should see GitHub in the list with:
   - Permissions granted
   - Date approved
   - Approved by (your admin account)

3. This audit trail helps you track:
   - What third-party apps have been approved
   - When approvals were made
   - By whom

4. Use this to regularly review whether all approved apps still make sense. If an app is no longer used, revoke its permissions.

---

## Lab 5.9.6 — Revoke User Consent (Revoke an Approval)

**Tools:** Entra admin center
**Prerequisite:** Lab 5.9.5 complete

If you approved an app and later realize it shouldn't have access, revoke the consent.

1. Navigate to **Applications** → **Permissions**.

2. Find the app you want to revoke (e.g., GitHub from Lab 5.9.4).

3. Click the **X** or **Revoke** button next to it.

4. Confirm revocation.

5. The app is now revoked. Users cannot sign in to it until you re-approve.

---

## The Scenario Check

Contoso's consent policy now requires admin approval for third-party apps. Users can still grant permissions to Microsoft apps, but anything external (GitHub, Slack, etc.) goes through Jordan Kim (IT admin) for review.

Morgan Chen tried to sign in to GitHub and got "admin consent required." Morgan requested approval. Jordan reviewed the request, saw that GitHub is legitimate, and approved it. Morgan can now sign in.

A month later, Contoso decides to move away from GitHub and use Azure DevOps instead. Jordan navigates to **Permissions**, finds GitHub, and revokes its access. GitHub can no longer sign in to Contoso's tenant. Any remaining permissions are cleaned up automatically.

New employees asking for access to apps go through the same workflow: request → admin review → approve/deny. This is enterprise identity governance.

---

## Check Your Understanding

- [ ] Navigate to **Consent and permissions** and identify the current tenant policy
- [ ] State what would happen if a user tried to sign in to a third-party app under the current policy
- [ ] Explain the difference between user consent and admin consent
- [ ] Describe what admins see when they review a consent request
- [ ] Create a scenario where you would use "Allow user consent for Microsoft only" (vs other policies)

---

## Common Mistakes

**Setting consent policy to "Allow user consent for all apps" in an enterprise.**
Users can now grant any app any permission. A malicious app tricks users into granting `User.ReadWrite.All`. The app modifies all user accounts. Your security posture is broken. Enterprise tenants should require admin consent for third-party apps.

**Not monitoring consent requests.**
Users request approval for apps. Requests pile up in **Admin consent requests** unanswered for weeks. Users think they're blocked; actually they're just waiting. Designate someone to check consent requests at least weekly.

**Approving consent without reviewing what the app requests.**
A user requests an app that says it needs "read all users and modify all users." You see the user's name and approve without reading the permissions. The app now has `User.ReadWrite.All`. Always review the specific permissions before approving.

**Forgetting to revoke permissions when apps are decommissioned.**
You stop using Slack, but its admin consent remains active. If someone gains Slack credentials, they can still connect to Contoso. Always revoke permissions from apps you're retiring.

---

## What's Next

Unit 5 is now complete. You can configure SSO, manage user access, audit permissions, and set up governance workflows. Unit 6 (Identity Security) shifts focus from "giving access safely" to "detecting and responding to attacks." You'll configure Identity Protection, read security logs, and practice incident response.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.8 — Automated Provisioning with SCIM](../5-8-provisioning-scim/5-8-provisioning-scim.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Unit 6: Identity Security →](../../UNIT%206%20-%20IDENTITY%20SECURITY/6-1-identity-protection-risk-detection/6-1-identity-protection-risk-detection.md)
