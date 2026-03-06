# Automated Provisioning with SCIM
*Create and update user accounts in SaaS apps automatically — no manual spreadsheets, no forgotten offboardings.*

> 🟡 **Intermediate** · Unit 5, Module 5.8 · ⏱️ 60 minutes

---

## What You Will Learn

- Explain what SCIM provisioning is and why it matters
- Understand the difference between SSO (sign-in) and provisioning (account creation)
- Configure SCIM provisioning for a SaaS app
- Set provisioning scope (which users get accounts)
- Map attributes from Entra ID to the SaaS app's user schema
- Monitor provisioning logs and diagnose sync failures

---

## Why This Matters

*SC-300: Plan and implement access management for applications — Configure automated user provisioning for applications*

SSO (Modules 5.3–5.5) means users can sign in to apps using Entra ID. But who has an account in the app in the first place? **Provisioning** automatically creates and maintains those accounts. Without provisioning, you manually add users to Slack one by one. With provisioning, Entra ID automatically creates Slack accounts for everyone in your scope. When you remove someone from Entra ID (they leave the company), provisioning automatically deactivates their Slack account. This is the enterprise way to manage SaaS users at scale.

---

## 🔄 SSO vs Provisioning: They Work Together

| | SSO | Provisioning |
|---|---|---|
| **What it does** | Lets user sign in using Entra ID credentials | Creates and maintains user accounts in the app |
| **When it happens** | At sign-in time | On a schedule (usually hourly) |
| **If configured alone** | User can sign in, but has no account in the app (error) | App has user account, but no way to sign in (SSO missing) |
| **Together** | User is created automatically, and signs in seamlessly | ✅ Complete solution |

**Example:** Without provisioning, you configure Slack SSO. Alex tries to sign in. Entra ID authenticates her and redirects her to Slack with a token. Slack says "I don't have an account for Alex" and denies access. Provisioning solves this: Entra ID pre-creates Alex's account in Slack before she ever tries to sign in.

---

## 📤 What SCIM Is

**SCIM = System for Cross-domain Identity Management**

SCIM is a standard protocol for provisioning. Entra ID periodically sends Slack (or any SCIM-supporting app) a feed of users:
- "Create account for Alex Lee, email alex.lee@contoso.com"
- "Update Morgan Chen's manager to Jordan Kim"
- "Disable account for Sam Taylor (no longer employed)"
- "Delete account for John Doe (left company 90 days ago)"

The app receives these commands via SCIM and applies them to its own user database.

---

## ⚙️ The Provisioning Process

**Step 1: Set provisioning scope**
- Decision: Which users should get accounts in the app?
- Option A: All users
- Option B: Users in a specific group (e.g., "Finance-Slack-Users")
- Option C: Users matching a filter (e.g., "Department = Finance")

**Step 2: Map attributes**
- Entra ID has: `user.displayName`, `user.mail`, `user.jobTitle`
- Slack expects: `name`, `email`, `title`
- Mapping: `user.displayName` → `name`, `user.mail` → `email`, etc.

**Step 3: Enable provisioning**
- Entra ID starts the provisioning service
- Service connects to Slack using a SCIM token
- Service runs provisioning cycles (usually hourly)

**Step 4: Monitor**
- Entra ID logs every user it created, updated, deleted
- You check logs to verify sync is working
- If a user fails to provision, logs show why

---

## Lab 5.8.1 — Enable SCIM Provisioning for Slack

**Time:** 20 minutes
**Tools:** Entra admin center, Slack admin settings
**Prerequisite:** Module 5.3 complete; Slack SSO is configured

Slack is a SCIM-supporting app. You'll enable provisioning from Entra ID.

**Step 1: Get Slack's SCIM endpoint and token**

1. Sign in to Slack as admin: `https://<workspace>.slack.com/admin`

2. Navigate to **Settings** → **Authentication** → **SCIM API** (or similar path).

3. Slack will show:
   - **SCIM endpoint URL** (looks like `https://api.slack.com/scim/v2/`)
   - **API token** (a secret used to authenticate Entra ID)

4. Copy both values. You'll paste them into Entra ID.

**Step 2: Enable provisioning in Entra ID**

5. Navigate to **Applications** → **Enterprise applications** → **Slack**.

6. On the left menu, click **Provisioning**.

7. Set **Provisioning Status** to **On**.

8. In the **Admin Credentials** section:
   - **Tenant URL:** Paste Slack's SCIM endpoint URL
   - **Secret Token:** Paste Slack's SCIM API token

9. Click **Test Connection**. Entra ID tests the token — if valid, you'll see "Success."

10. Click **Save**.

Now Entra ID can connect to Slack for provisioning.

---

## Lab 5.8.2 — Set Provisioning Scope

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.8.1 complete

Decide which users should be provisioned to Slack.

1. On the **Provisioning** page, find **Scope** section.

2. You'll see options:
   - **Sync only assigned users and groups** — Only users/groups explicitly assigned to Slack app
   - **Sync all users and groups** — Everyone in the tenant (not recommended for selective apps)

3. Select **Sync only assigned users and groups**.

4. This means: Only users and groups you've assigned to Slack (from Module 5.6) will be provisioned.

5. Click **Save**.

---

## Lab 5.8.3 — Map User Attributes

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.8.2 complete

Attribute mapping tells Entra ID how to translate its user data into Slack's format.

1. On the **Provisioning** page, find **Mappings** section.

2. Click **Provision Azure Active Directory Users** (or similar).

3. You'll see a list of mappings. The default includes:
   - `userPrincipalName` → `userName` (Slack username)
   - `displayName` → `name` (Slack display name)
   - `mail` → `emails[primary eq true].value` (Slack email)
   - `externalId` → `id` (Slack unique ID)

4. These defaults are usually correct for Slack. Do not change them unless Slack's documentation specifies different mappings.

5. Scroll down to see optional mappings:
   - `jobTitle` → `title` (optional, if you want job titles in Slack)
   - `department` → `department` (optional, if Slack uses departments)

6. If you want to map job title, click the mapping and set it:
   - Source: `jobTitle`
   - Target: `title`

7. Click **Save**.

---

## Lab 5.8.4 — Start Provisioning Cycle and Monitor Logs

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.8.3 complete

1. On the **Provisioning** page, find **Settings** section.

2. Under **Provisioning cycle frequency**, you'll see options:
   - **On demand** — Only when you manually trigger
   - **Scheduled** — Every 40 minutes or hour (default)

3. Leave as scheduled (default).

4. Click **Save**.

5. Provisioning should start automatically. To manually trigger it:
   - Click **Start provisioning** button if available

6. Wait 2-3 minutes for the first cycle to complete.

7. Navigate to **Provisioning** → **Provisioning logs** to see what happened.

8. You should see entries like:
   - "Create: Alex Lee (alex.lee@contoso.com)"
   - "Create: Morgan Chen (morgan.chen@contoso.com)"
   - "Create: Jordan Kim (jordan.kim@contoso.com)"
   - "Skip: Sam Taylor (not assigned to app)"

9. For each user, check the **Status**:
   - **Success** → Account created in Slack
   - **Failure** → Account creation failed; see the error
   - **Skipped** → User doesn't match scope (not assigned, not in filtered group, etc.)

10. If any failures, read the error message. Common issues:
    - **Email already exists** → That user already has a Slack account; Entra ID skips it
    - **Invalid email format** → The email attribute is malformed; fix the user's email in Entra ID
    - **API token invalid** → The SCIM token is wrong or expired; update it

---

## Lab 5.8.5 — Test Deprovisioning (Remove User from Scope)

**Time:** 10 minutes
**Tools:** Entra admin center, Slack
**Prerequisite:** Lab 5.8.4 complete; provisioning has run at least once

Test that when you unassign a user from Slack, their account is deactivated in Slack.

1. Navigate to **Applications** → **Enterprise applications** → **Slack** → **Users and groups**.

2. Find **Sam Taylor** (the unassigned user from Module 5.6).

3. Click **...** next to Sam's assignment (if assigned) or note that Sam is not assigned.

4. If Sam is assigned, click **Remove assignment**. Sam is now unassigned.

5. Wait for the next provisioning cycle (up to 40 minutes, or manually trigger if available).

6. Check **Provisioning logs** again.

7. You should see an entry like:
   - "Disable: Sam Taylor (sam.taylor@contoso.com)" — Sam's account in Slack is deactivated

8. Sam can no longer sign in to Slack (assuming Slack respects the deprovisioning signal).

9. This is the power of provisioning: when someone leaves or changes roles, Slack accounts are automatically managed.

---

## The Scenario Check

Contoso has configured SCIM provisioning for Slack. Entra ID automatically created accounts for Alex Lee, Morgan Chen, Jordan Kim, and River Patel in Slack. Their email addresses, names, and job titles are synced from Entra ID to Slack.

When Contoso hired a new Finance analyst (not yet created in this series), the admin adds them to the Entra ID tenant and assigns them to Slack. Within the next provisioning cycle (typically an hour), Slack automatically creates their account. No manual Slack admin action needed.

When an employee leaves Contoso, the admin removes them from Entra ID. The next provisioning cycle deactivates their Slack account. Their messages remain, but they can no longer sign in.

Provisioning logs are monitored by Jordan Kim (IT admin) monthly. If any users fail to provision, the logs show why, and Jordan fixes the issue.

---

## Check Your Understanding

- [ ] Navigate to **Provisioning** and confirm the status is **On** and scope is set correctly
- [ ] Find an attribute mapping and explain what it does
- [ ] Open **Provisioning logs** and find at least three successful provisions
- [ ] Describe what happens to a user's app account when you remove the user from Entra ID
- [ ] Explain the difference between SSO and provisioning in one sentence

---

## Common Mistakes

**Not setting provisioning scope to "assigned users only".**
You enable provisioning with scope set to "all users." Every user in your tenant gets a Slack account, whether you intended it or not. Soon you have 500 Slack users when you only wanted 50. Always set scope to "assigned users and groups."

**Mapping user attributes incorrectly.**
You map `jobTitle` to a field Slack doesn't recognize. Slack ignores the mapping. You think it didn't work, but actually it works fine — Slack just doesn't use job titles. Only map attributes that the app documents as supported.

**Forgetting to enable provisioning after setting it up.**
You configure everything perfectly but forget to set **Provisioning Status** to **On**. Users are never created. No error, no warning — provisioning just silently doesn't happen. Always verify the status toggle is **On**.

**Using an old SCIM token.**
You enable provisioning with a Slack SCIM token. Six months later, Slack rotates the token (for security). Provisioning suddenly fails. The error message says "API token invalid." Generate a new token from Slack and update Entra ID.

**Deprovisioning (deleting accounts) before testing thoroughly.**
You enable deprovisioning, which removes accounts from Slack when users are unassigned. You test by unassigning a user. Their Slack account is deleted — including their message history (if Slack is configured to delete on deprovisioning). Be careful with deprovisioning settings; test first, enforce later.

---

## What's Next

Module 5.9 covers **Consent Settings and Admin Consent Workflow** — how to control when users can grant permissions to apps, and how to set up an approval workflow so admins review and approve app permissions before they're granted. This completes Unit 5 by tying together SSO, provisioning, and governance.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.7 — App Permissions and Admin Consent](../5-7-app-permissions-consent/5-7-app-permissions-consent.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.9 — Consent Settings and Admin Consent Workflow →](../5-9-consent-workflow/5-9-consent-workflow.md)
