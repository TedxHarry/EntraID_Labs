# Configuring PIM — Make Admin Access Controlled
*Set up which roles are eligible, which require approval, and what users must do to activate them.*

> 🟡 **Intermediate** · Unit 7, Module 7.2 · ⏱️ 60 minutes

---

## What You Will Learn

- Convert permanent admin assignments to eligible assignments
- Configure role settings (time limits, approval requirements, MFA)
- Set up notification rules
- Create approval workflows
- Design a PIM strategy for your tenant

---

## Why This Matters

*SC-300: Plan and automate identity governance — Configure PIM role settings and policies*

Module 7.1 explained *why* PIM matters. This module shows you *how to set it up*. Configuration is where security policy becomes operational. Wrong settings can either make PIM unusable (too much friction) or ineffective (too weak). This module teaches you to balance security and usability.

---

## 🔧 Three Configuration Areas

### Area 1: Role Eligibility Management
**Goal:** Decide who can become admin (eligible), and remove permanent admins.

| Task | What to Configure | When |
|---|---|---|
| **Add eligible users** | User A can activate role X | Onboarding |
| **Remove permanent admins** | User B is no longer admin permanently | Migration to PIM |
| **Set duration** | Activation lasts 2 hours (then expires) | Creating eligibility |

---

### Area 2: Role Settings
**Goal:** Configure how activation works (approval required? MFA required? Max duration?).

| Setting | Options | Example |
|---|---|---|
| **Require approval** | Yes / No | Global Admin requires approval, User Admin doesn't |
| **Approvers** | Specific users or groups | River Patel, or "All admins" |
| **Max activation duration** | 1–8 hours (you set limit) | 2 hours (user can't activate longer) |
| **Require MFA** | Yes / No | Global Admin requires MFA, others don't |
| **Conditional Access** | Apply CA policies | Block activation from non-corporate devices |
| **Notifications** | Alert admins of activations | "Jordan activated Global Admin" |

---

### Area 3: Approval Workflow
**Goal:** Who approves activation requests, and how.

| Element | Decision | Example |
|---|---|---|
| **Approver role** | Who can approve? | Security admin, or specific users |
| **Notification** | How do approvers get notified? | Email alert when request arrives |
| **Response time** | Deadline for approval | 24 hours (after that, auto-deny?) |
| **Escalation** | What if approver is out? | Escalate to secondary approver |

---

## Lab 7.2.1 — Convert Permanent Admin to Eligible Assignment

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 7.1 complete; you identified a permanent admin in Lab 7.1.3

Goal: Take a current permanent admin and make them eligible instead, removing permanent access.

### Step 1: Identify Target Role

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles**.

2. Click on **Global Administrator** (or another high-privilege role).

3. You'll see two tabs:
   - **Active assignments** — Permanent admins (right now)
   - **Eligible assignments** — Eligible admins (can activate)

4. Review **Active assignments**. You should see at least one user (maybe just yourself if you're the primary admin).

5. **Choose a test user** — ideally not yourself (so you can test activation without affecting yourself).
   - If you only see yourself: Create a test user first (or use Jordan Kim from the scenario)

---

### Step 2: Add User to Eligible Assignment

1. On the **Global Administrator** role page, find the **Eligible assignments** section.

2. Click **+ Add assignment** (or **+ Add eligible assignment**).

3. **Select user:** Choose a test user (example: Jordan Kim).

4. **Duration:** Set **Assignment type** to **Eligible** (not Permanent).

5. **Duration start/end (optional):** You can set an expiration date for the eligibility itself (example: "Jordan is eligible until 2025-12-31"). Leave blank for permanent eligibility.

6. Click **Assign** (or **Next** → **Assign**).

**Result:** The user is now eligible for the role. They can request activation, but aren't admin by default.

---

### Step 3: Remove Permanent Assignment

1. Go back to the **Active assignments** tab for **Global Administrator**.

2. Find the user you want to remove (e.g., a test account or secondary admin).

3. Click **...** (menu) → **Remove** or click the user → **Remove assignment**.

4. Confirm: "Remove permanent assignment?"

5. Click **Remove**.

**Result:** User is no longer a permanent admin. If they had eligibility added in Step 2, they can still become admin by activating. If not, they have no admin access at all.

> ⚠️ **Warning:** Don't remove the only Global Administrator from permanent access! You'll lock yourself out. Ensure there's at least one other Global Administrator (permanent or eligible) before removing the last one. In production, set up break-glass accounts first (Module 4.1).

---

## Lab 7.2.2 — Configure Role Settings (Approval, MFA, Time Limits)

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.2.1 complete

Configure how activation works for a high-privilege role.

### Step 1: Open Role Settings

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles**.

2. Click on **Global Administrator**.

3. On the left menu, find **Settings** (or click the role and look for **Role settings**, **Configure**, or **Edit**).

4. You'll see settings for activating this role.

---

### Step 2: Configure Approval

1. Find setting: **Require approval to activate this role** (or **On activation, require**).

2. Set to **Yes** (for high-privilege roles like Global Admin, approval is essential).

3. Under **Select approvers**, choose who can approve Global Admin activations:
   - Option A: **Specific users** → Select River Patel, Alex Lee (individual admins)
   - Option B: **All admins in this role** → Any Global Admin can approve requests from others
   - Option C: **Specific role** → "All members of Cloud Application Administrator can approve"

4. For this lab, select **Specific users** → **River Patel** (as the approver).

5. Click **Next** (or continue to next setting).

---

### Step 3: Configure MFA

1. Find setting: **Require Azure MFA on activation** (or **Require MFA**).

2. Set to **Yes** (before activating, user must complete MFA again for verification).

3. This means: Even if Jordan is already signed in, she must complete MFA (phone verification) again before admin access is granted.

4. Click **Next**.

---

### Step 4: Configure Time Limits

1. Find setting: **Active assignment maximum duration** (or **Duration**).

2. Set the maximum duration a user can activate for:
   - Example: **2 hours** (user cannot request more than 2 hours)
   - Users can request shorter durations (1 hour, 30 minutes), but not longer

3. If you want to enforce a specific duration (no choice), set **Eligible assignment duration** (or **Eligible assignment expiry**) to a fixed number.

4. Click **Next**.

---

### Step 5: Configure Notifications

1. Find setting: **Send notifications when members are assigned this role** (or **Notifications**).

2. Options:
   - **Admin activation notification:** Send email to admins when someone activates this role
   - **Approver notification:** Send email to approvers when there's a request to approve

3. Enable both:
   - When Jordan activates Global Admin, you (admin) get notified
   - When Jordan requests activation, River (approver) gets notified

4. Click **Save** (or **Update**).

**Result:** Global Administrator role now requires approval, MFA, has a 2-hour max duration, and generates notifications.

---

## Lab 7.2.3 — Configure Settings for a Low-Privilege Role

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.2.2 complete

Demonstrate that not all roles need strict controls. Configure a lower-risk role.

### Step 1: Select a Lower-Risk Role

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles**.

2. Click on **User Administrator** (a lower-risk role — can create/manage users but not global settings).

---

### Step 2: Configure Less Restrictive Settings

1. Open **Settings** for **User Administrator**.

2. Configure:
   - **Require approval:** **No** (lower-risk role doesn't need approval)
   - **Require MFA:** **Yes** (keep MFA for verification, but skip approval)
   - **Max activation duration:** **4 hours** (can be longer than Global Admin since it's lower risk)
   - **Notifications:** Enable (still want to know when User Admin is activated)

3. **Rationale:** User Administrator is useful but less dangerous than Global Admin. Approvals slow down admin work unnecessarily. MFA is sufficient for verification.

4. Click **Save**.

**Result:** Different roles have different controls based on risk. High-privilege → strict. Medium-privilege → moderate. Low-privilege → lighter controls.

---

## Lab 7.2.4 — Review PIM Settings (Tenant-Wide Defaults)

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.2.3 complete

Check global PIM settings that apply across all roles.

1. Navigate to **Privileged Identity Management** → **Settings** (top-level settings, not individual roles).

2. You'll see options:
   - **Alert settings** — When to alert admins of suspicious activity
   - **Approval settings** — Default approval rules
   - **Notification settings** — How admins are notified

3. Review:
   - **Alert if activation occurs after hours** — Yes (alert if Global Admin is activated at 2 AM)
   - **Alert if multiple activations from same user** — Yes (if one user requests Global Admin 10 times in a day, alert)
   - **Require reason for activation** — Yes (users must provide a reason: "Password reset for departing employee")

4. Click **Save** if you make changes.

---

## Lab 7.2.5 — Set Up a Multi-Level Approval Chain

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.2.4 complete

Advanced scenario: Require multiple approvers for critical roles.

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles** → **Global Administrator** → **Settings**.

2. Find **Approvers** section.

3. Set multiple approvers:
   - Primary approver: **River Patel** (HR Manager, admin)
   - Secondary approver: **Alex Lee** (Finance, also admin)

4. Approval rule:
   - If set to "All approvers must approve": Both River and Alex must approve Global Admin requests
   - If set to "Any approver can approve": River or Alex can approve (faster)

5. For this lab, set to **"Any approver can approve"** (more practical).

6. Click **Save**.

**Result:** If River is out of office, Alex can approve activation requests. No bottleneck.

---

## The Scenario Check

Contoso has configured PIM:

**Global Administrator role:**
- Permanent admins: None (converted to eligible)
- Eligible admins: Jordan Kim, Alex Lee
- Approval required: Yes (River Patel is approver)
- MFA required: Yes
- Max duration: 2 hours
- Notifications: Enabled

**User Administrator role:**
- Eligible admins: River Patel, Morgan Chen
- Approval required: No (lower risk)
- MFA required: Yes
- Max duration: 4 hours
- Notifications: Enabled

**Result:** Admins can do their jobs without bottlenecks. High-risk tasks require approval and are audited. Lower-risk tasks are faster. No one is admin all the time.

Example: Jordan needs to reset a user's password.
- 2:00 PM: Jordan requests "Global Admin for 1 hour" (reason: "Password reset for Alex Lee")
- 2:02 PM: River gets notification, reviews request, approves it
- 2:03 PM: Jordan activates (completes MFA)
- 2:04 PM: Jordan is Global Admin, resets the password
- 3:04 PM: Access expires automatically

---

## Check Your Understanding

- [ ] Explain the difference between "Active" and "Eligible" role assignments
- [ ] Describe why approval is required for high-privilege roles but not low-privilege ones
- [ ] State what happens when a role activation time limit is reached
- [ ] Explain why MFA is required before activation (even if user is already signed in)
- [ ] Design a PIM configuration for your tenant's admin roles

---

## Common Mistakes

**Requiring approval for all roles.**
You set up PIM with approval required for every role. Suddenly, a user needs to add a group member (User Admin role). Activation request goes to approver. Approver is in a meeting. Request sits in queue for 30 minutes. Users complain: "I can't do my job." Set approval only for high-privilege roles (Global Admin, Security Admin). Allow low/medium roles to activate without approval.

**Setting max duration too short.**
You set Global Admin max duration to 30 minutes. An admin is mid-task (migrating users) when access expires. Admin has to request activation again, wait for approval, and restart the task. Set reasonable durations based on common tasks: 2 hours for password resets, 4 hours for configuration changes, 8 hours for large migrations.

**Not requiring MFA for activation.**
You configure PIM but don't require MFA for activation. An attacker with stolen admin credentials can activate roles without second-factor verification. Always require MFA for high-privilege role activation.

**Forgetting to add eligible users before removing permanent admins.**
You remove a permanent admin without making them eligible first. They lose all admin access. Now they can't even request activation. Always add users to eligible assignments *before* removing permanent assignments.

**Ignoring approval requests.**
A user requests Global Admin activation. The approver doesn't see the notification (email went to spam). Request expires unanswered. User thinks they're locked out, but really the approver just didn't see it. Monitor notifications closely. Set up a second approver as backup.

**Not testing PIM before enforcing it.**
You configure PIM on production roles without testing first. Admins can't activate. It takes 30 minutes to fix because approval workflow was misconfigured. Test on non-critical roles first. Get feedback from admins. Then expand to high-privilege roles.

---

## What's Next

Module 7.3 covers **Using PIM — Just-in-Time Access** from the user's perspective. You'll practice requesting activation, waiting for approval, completing MFA, and then using admin access. This is what users will do every day once PIM is enabled.

---

> 📚 **Entra ID — Zero to Admin** · Unit 7: Privileged Access Management
>
> [← Module 7.1 — PIM Overview](../7-1-pim-overview/7-1-pim-overview.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 7.3 — Using PIM (Just-in-Time Access) →](../7-3-pim-activation/7-3-pim-activation.md)
