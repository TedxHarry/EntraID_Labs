# User Assignment — Control Who Can Access an App
*SSO means everyone can sign in. User assignment means only the right people can.*

> 🟢 **Beginner** · Unit 5, Module 5.6 · ⏱️ 45 minutes

---

## What You Will Learn

- Explain the difference between "assignment required" and open access
- Assign individual users and groups to an app
- Understand why group assignment scales better than individual assignment
- Apply Conditional Access policies to specific apps
- Test access as both an assigned and unassigned user

---

## Why This Matters

*SC-300: Implement access management for applications — Manage user and group assignments to applications*

Modules 5.1–5.5 taught *how to set up SSO*. This module teaches *who gets to use the SSO*. Without user assignment, anyone in your Contoso tenant can sign in to Slack (assuming they know it exists). With user assignment, only the people you explicitly allow can access it. This is the difference between "available" and "controlled."

---

## 🔒 Assignment Required vs Open Access

**Without Assignment Required (Open Access):**
- Any user in the tenant can sign in to the app
- The app doesn't check if the user is "allowed"
- Good for: Internal communication tools (company chat, intranet), expense reporting
- Risk: Users sign up for apps you didn't know existed

**With Assignment Required (Controlled Access):**
- Only users/groups you explicitly assign can sign in
- Anyone else who tries gets "You do not have access"
- Good for: Sensitive apps (HR systems, financial software), paid SaaS (limit licenses)
- Risk: You must maintain the assignment list manually (or use group-based assignment)

**Rule of thumb:** Enterprise apps (Salesforce, ServiceNow) need assignment required. Internal tools can be open. Always require assignment for sensitive or licensed apps.

---

## 👥 Assigning Users vs Groups

| Approach | Method | Scaling | Maintenance |
|---|---|---|---|
| **Individual users** | Select each user one by one | Doesn't scale (50 users = 50 assignments) | High burden; every hire needs manual addition |
| **Groups** | Assign an entire group at once | Scales beautifully (add user to group, they get access) | Low burden; group membership drives access |

**Example:**
- **Bad:** Assign all 50 Finance users to Concur (expense app) individually. When you hire person 51, you manually add them. ❌
- **Good:** Create a group "Finance-All," assign that group to Concur. Add all Finance users to the group. New hire gets added to the group by HR automation; access is automatic. ✅

---

## Lab 5.6.1 — Enable Assignment Required on Slack

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 5.5 complete; Slack is configured with SSO

1. Navigate to **Applications** → **Enterprise applications** → **Slack**.

2. On the left menu, click **Properties**.

3. Find the setting **User assignment required?**

4. Toggle to **Yes**.

5. Click **Save**.

Now only users explicitly assigned to Slack can access it. Anyone else who tries to sign in will see: "You do not have access to this application."

---

## Lab 5.6.2 — Assign Users to the App

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.6.1 complete

1. Navigate to **Applications** → **Enterprise applications** → **Slack**.

2. On the left menu, click **Users and groups**.

3. Click **+ Add user/group**.

4. In the panel that opens, search for **Alex Lee**. Click to select.

5. Under **Select a role** (if shown), leave as default or select "User."

6. Click **Assign**.

7. Alex Lee now appears in the **Users and groups** list with status **Assigned**.

8. Repeat the process to assign **Morgan Chen**, **Jordan Kim**, and **River Patel**.

9. **Do not assign Sam Taylor** — you'll use Sam as the test case for access denial.

> 💡 **Tip:** Notice there's an option to assign a **Group** instead of a user. Click **+ Add user/group** again and select a group (e.g., "Finance-All") to assign the entire group at once. This is the scalable approach.

---

## Lab 5.6.3 — Test Access: Assigned vs Unassigned User

**Time:** 15 minutes
**Tools:** InPrivate browser, Slack workspace
**Prerequisite:** Lab 5.6.2 complete

**Test 1: Assigned user (Alex Lee)**

1. Open an **InPrivate** browser window.

2. Navigate to your Slack workspace: `https://<workspace>.slack.com`

3. Click the SSO sign-in button.

4. Sign in as **alex.lee@yourdomain.onmicrosoft.com** with her password.

5. Entra ID authenticates her. You're redirected back to Slack.

6. **Expected result:** Alex is signed in and can access Slack. The assignment worked.

7. Note the Slack workspace URL in the address bar — you're inside the workspace.

8. Sign out of Slack.

**Test 2: Unassigned user (Sam Taylor)**

9. In the same InPrivate window, navigate to Slack again: `https://<workspace>.slack.com`

10. Click the SSO sign-in button.

11. Sign in as **sam.taylor@yourdomain.onmicrosoft.com** with Sam's password.

12. Entra ID authenticates Sam.

13. **Expected result:** You're redirected back to Slack, but you see an error: "You do not have access to this application" or "Access denied."

14. This is the correct behavior — Sam is not assigned, so assignment required blocks access.

15. Verify this in the Entra ID sign-in logs:
    - Navigate to **Monitoring** → **Sign-in logs** (in admin account)
    - Find Sam's sign-in attempt
    - Check the **Conditional Access** column
    - You should see a policy result (likely not applicable, but check Application assignment status if logged)

**You'll know this worked when:**
- Alex can sign in
- Sam cannot sign in
- The difference is clear: assignment controls access

---

## Lab 5.6.4 — Assign a Group Instead of Individual Users

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 5.6.3 complete

You've assigned individual users. Now see the scalable approach: group assignment.

1. Navigate to **Identity** → **Groups** → **All groups** → **+ New group**.

2. **Group type:** Security

3. **Group name:** `Slack-Users`

4. **Group description:** `Users with Slack access`

5. **Members:** Add Alex Lee, Morgan Chen, and Jordan Kim

6. Click **Create**.

7. Navigate to **Applications** → **Enterprise applications** → **Slack** → **Users and groups**.

8. Click **+ Add user/group**.

9. Search for **Slack-Users** group. Select it.

10. Click **Assign**.

11. Now the group "Slack-Users" appears in the assignment list. All three members (Alex, Morgan, Jordan) have access through group membership.

12. **Benefit:** Add a new user to the Slack-Users group, they automatically get Slack access. Remove someone from the group, they lose access. No manual app assignments needed.

---

## Lab 5.6.5 — Apply Conditional Access to an App

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 4.8 (Unit 4) complete; CA policies exist

You can apply Conditional Access policies to specific apps. For example: "Slack access requires MFA" or "Slack requires a compliant device."

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. Click **+ New policy**.

3. **Name:** `Require MFA for Slack`

4. **Assignments:**
   - **Users and groups:** Select **All users**
   - **Target resources:** Click **Select what this policy applies to** → **Cloud apps and actions** → Click **Select apps**
   - Search for **Slack** and select it
   - Click **Select**

5. **Access control** (Grant):
   - Click **Grant**
   - Check **Require multifactor authentication**
   - Click **Select**

6. **State:** **Report-only** (test before enforcing)

7. Click **Create**.

Now any attempt to sign in to Slack will be evaluated against this policy. In Report-Only mode, access is still granted but the policy is logged. When you switch to Enforce, users will be prompted for MFA before accessing Slack.

---

## The Scenario Check

Contoso's Slack is now protected by user assignment. Alex Lee, Morgan Chen, Jordan Kim, and River Patel are assigned and can sign in. Sam Taylor (the contractor) is not assigned and cannot access Slack — even though Sam has valid Contoso credentials. This is intentional — contractors should not have access to internal communication tools without explicit permission.

A Conditional Access policy requiring MFA for Slack is in Report-Only mode, documenting what would happen when enforced.

When Contoso hires new Finance employees, the admin adds them to the "Slack-Users" group. They automatically get Slack access the next time they sign in — no manual app assignment needed.

---

## Check Your Understanding

- [ ] Navigate to **Slack** → **Properties** and confirm "User assignment required?" is set to **Yes**
- [ ] In **Users and groups**, count how many users and groups are assigned
- [ ] Create a new group (`Sales-Slack`), add one user to it, and assign the group to Slack
- [ ] Sign in as an unassigned user and verify access is denied
- [ ] Explain why group assignment is better than individual assignment at scale

---

## Common Mistakes

**Leaving "Assignment required?" as No on sensitive apps.**
You configure Slack SSO but forget to enable assignment required. Any employee can sign up for Slack without permission. Suddenly 200 people have Slack access (and your company is paying per user). Always require assignment for paid or sensitive apps.

**Assigning individual users when groups exist.**
You manually assign 50 employees to Concur. Months later, you hire person 51 and forget to add them. They can't access Concur and create a support ticket. Meanwhile, an employee who left three months ago still has access because you forgot to unassign. Groups prevent this — add/remove from the group, access follows automatically.

**Removing user assignments without removing group assignments.**
You assign John individually AND add John to the "Finance-Slack" group. You later unassign John thinking he no longer has access. He still has Slack access through the group. When removing someone, check both individual assignments and group membership.

**Expecting assignment to propagate immediately.**
You assign a user to an app at 2pm. The user tries to sign in at 2:01pm and gets "access denied." Assignments can take a few minutes to propagate. Wait 5 minutes and retry. If still denied, check the sign-in logs.

---

## What's Next

Module 5.7 covers **App Permissions and Admin Consent** — the difference between delegated permissions (app acts on behalf of user) and application permissions (app acts as itself). You'll audit what permissions apps have been granted and understand the security risks of over-permissioned apps.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.5 — OIDC/OAuth SSO](../5-5-oidc-oauth-sso/5-5-oidc-oauth-sso.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.7 — App Permissions and Admin Consent →](../5-7-app-permissions-consent/5-7-app-permissions-consent.md)
