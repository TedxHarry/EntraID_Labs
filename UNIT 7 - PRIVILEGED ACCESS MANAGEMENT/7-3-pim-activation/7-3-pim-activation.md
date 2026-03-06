# Using PIM — Just-in-Time Access
*You need admin access now. Request it, get approved, activate it, and use it. Then it expires.*

> 🟡 **Intermediate** · Unit 7, Module 7.3 · ⏱️ 50 minutes

---

## What You Will Learn

- Request role activation through PIM
- Complete MFA to verify identity before activation
- Monitor activation requests (as an approver)
- Activate approved requests
- Understand the admin access window (when it's active, when it expires)

---

## Why This Matters

*SC-300: Plan and automate identity governance — Use PIM for just-in-time admin access*

Module 7.2 explained *how to configure* PIM. This module shows the *end-user experience*. Admins need to know:
- How to request activation (can't do their job without it)
- How long approval takes (planning timelines)
- How to activate once approved (the actual process)
- What to do when access expires (request again or plan ahead)

Understanding the activation flow makes you better at PIM governance — you can set realistic timelines and requirements.

---

## 🔄 The Activation Flow

```
Admin needs role
    ↓
Admin requests activation in PIM
    ↓
If approval required:
  Admin waits for approver
    ↓
  Approver reviews request
    ↓
  Approver approves or denies
    ↓
  If approved: Continue
  If denied: User is notified (end)
    ↓
Admin activates role (from PIM UI)
    ↓
Admin completes MFA
    ↓
Admin role is now active (until expiration)
    ↓
Admin performs admin tasks
    ↓
Access expires (or admin manually deactivates)
```

---

## Lab 7.3.1 — Request Role Activation (User Perspective)

**Time:** 15 minutes
**Tools:** Entra admin center, user identity
**Prerequisite:** Module 7.2 complete; you've set up eligible admin assignments

Scenario: You are Jordan (IT Admin, eligible for Global Admin but not active). You need to reset a user's password. You must request activation first.

### Step 1: Sign In As Eligible User

1. Open an **InPrivate** browser window.

2. Navigate to `portal.azure.com` (or `myapps.microsoft.com`).

3. Sign in as **jordan.kim@yourdomain.onmicrosoft.com** (your test admin user).

4. You're now signed in as Jordan (not as admin, since eligibility is not active).

---

### Step 2: Navigate to PIM

1. In the Azure portal (or Entra admin center), search for **"Privileged Identity Management"** or navigate to **Identity Governance** → **Privileged Identity Management**.

2. Click **My roles** (or **Azure AD roles** → **My roles**).

3. You'll see two tabs:
   - **Eligible roles** — Roles you can activate (e.g., Global Administrator)
   - **Active roles** — Roles you're currently admin for (should be empty if not activated)

4. Review **Eligible roles**. You should see **Global Administrator** (since you made yourself/Jordan eligible in Lab 7.2.1).

---

### Step 3: Request Activation

1. In **Eligible roles**, find **Global Administrator**.

2. Click **Activate** (or **Request activation**, or **...** → **Activate**).

3. A form appears: **Activation request**

4. Fill in the form:
   - **Duration:** How long do you need access? (example: **1 hour**)
     - This is limited by max duration configured in Lab 7.2.2 (e.g., 2 hours max)
     - You can request less (30 minutes), but not more (max 2 hours)
   - **Reason:** Why do you need access? (example: **"Password reset for departing employee"**)
     - Optional but encouraged (audit trail is clearer)
   - **Request for:** Yourself (not delegate, just you)

5. Click **Activate** (or **Request**).

**Result:** Your activation request is submitted. If approval is required (configured in Lab 7.2.2), the request goes to approver (River Patel). If no approval required, activation might be immediate.

---

## Lab 7.3.2 — Monitor Activation Request (As Approver)

**Time:** 10 minutes
**Tools:** Entra admin center, approver identity
**Prerequisite:** Lab 7.3.1 complete; you've submitted an activation request

Scenario: You are River (HR Manager, configured as approver in Lab 7.2.2). Jordan's activation request arrives. You must review and approve it.

### Step 1: Sign In As Approver

1. Open a **new InPrivate** browser window.

2. Sign in as **river.patel@yourdomain.onmicrosoft.com** (the approver).

3. You're now River.

---

### Step 2: Navigate to PIM Approvals

1. Navigate to **Privileged Identity Management** → **Approvals** (or **Approve requests**, or **Pending approvals**).

2. You'll see a list of pending activation requests (if approval is configured and requests exist).

3. Look for Jordan's request: **"Global Administrator activation — Requested by Jordan Kim"**

4. Click on it to see details:
   - **User:** Jordan Kim
   - **Role:** Global Administrator
   - **Duration requested:** 1 hour
   - **Reason:** "Password reset for departing employee"
   - **Time requested:** [timestamp]

---

### Step 3: Approve the Request

1. On the request details page, you'll see buttons: **Approve** and **Deny**.

2. Review the request:
   - Is the user Jordan Kim? (expected ✓)
   - Is the reason legitimate? (password reset ✓)
   - Is the duration reasonable? (1 hour for password reset ✓)

3. Click **Approve**.

4. You may be asked to provide a comment: "Why are you approving this?"
   - Example: "Legitimate password reset task" (optional)

5. Click **Approve** (confirm).

**Result:** Jordan's request is approved. Jordan will now see the activation is ready to complete (in her PIM "My roles" page).

---

## Lab 7.3.3 — Complete Activation (MFA)

**Time:** 10 minutes
**Tools:** Entra admin center, user identity, phone for MFA
**Prerequisite:** Lab 7.3.2 complete; request has been approved

Back to Jordan's perspective: The request was approved. Now you must activate it.

### Step 1: Return to PIM (As Jordan)

1. Go back to the **InPrivate** browser window where you're signed in as Jordan.

2. Navigate to **Privileged Identity Management** → **My roles**.

3. You should now see the role in a different state:
   - **Before approval:** "Activate" button (pending approval)
   - **After approval:** "Activate" button available, or status shows "Approved"

---

### Step 2: Activate the Approved Request

1. Click the activation that was just approved.

2. Click **Activate** or **Confirm activation** (button label varies).

3. You'll be prompted for **MFA verification**:
   - Method: "Approve sign-in on my authenticator app" or "Send code to my phone"
   - Your authenticator app receives a notification, or your phone receives a text with a code

4. **Complete MFA:**
   - If using authenticator: Approve the notification in the Microsoft Authenticator app
   - If using SMS: Enter the code sent to your phone

5. Click **Verify** or **Confirm**.

**Result:** MFA is complete. Your activation is now active.

---

## Lab 7.3.4 — Verify Active Admin Access

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.3.3 complete; activation is active

Verify that you now have admin access and can perform admin tasks.

### Step 1: Check "My roles" Status

1. In **Privileged Identity Management** → **My roles**, find **Global Administrator**.

2. Status should now show:
   - **Active** (with a timer: "Expires in 59 minutes" — if you requested 1 hour)
   - Green indicator or "Active" label

3. The role is now active.

---

### Step 2: Test Admin Access

1. Navigate to **Users** → **All users** (or **Groups**, or another admin feature).

2. Try a basic admin action:
   - Example: Click on a test user, view their details, try to reset their password
   - This action should *succeed* because you're now Global Admin

3. If you weren't Global Admin, this action would show "You don't have permission."

**Result:** You've confirmed admin access is now active.

---

### Step 3: Monitor Active Role Duration

1. Return to **Privileged Identity Management** → **My roles** → **Active roles** (the tab).

2. Your **Global Administrator** is listed with a timer: "Expires in 48 minutes" (example, counting down from 1 hour).

3. This timer is live — it counts down in real-time.

---

## Lab 7.3.5 — Deny an Activation Request (As Approver)

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Previous labs complete; set up another test activation request

Scenario: An unusual activation request arrives. You deny it.

### Step 1: Request Another Activation (As Jordan, Different Role)

1. As Jordan, go to **Privileged Identity Management** → **My roles** → **Eligible roles**.

2. (If you configured another eligible role for Jordan, click it)

3. Request activation with an **unusual reason:**
   - Example: **Duration:** 8 hours (max allowed)
   - Example: **Reason:** "Testing"
   - Example: Time: **2:00 AM** (off-hours)

4. Submit the request.

---

### Step 2: Deny the Request (As River, Approver)

1. As River, go to **Privileged Identity Management** → **Approvals**.

2. Find the new request from Jordan.

3. Review it:
   - Duration: 8 hours (seems long)
   - Reason: "Testing" (vague)
   - Time: 2 AM (unusual)
   - Verdict: Deny (suspicious)

4. Click **Deny**.

5. Provide a reason (optional):
   - Example: "Off-hours activation request and vague reason. Please re-submit during business hours with specific purpose."

6. Click **Deny** (confirm).

**Result:** The request is denied. Jordan sees it in her activation history as "Denied."

---

## Lab 7.3.6 — Manual Deactivation (Early)

**Time:** 5 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.3.4 complete; you have active admin role

Scenario: You've completed your admin task early. You want to deactivate your role early to reduce the risk window.

### Step 1: Deactivate Active Role

1. As Jordan, go to **Privileged Identity Management** → **My roles** → **Active roles**.

2. You see **Global Administrator** listed with "Expires in 45 minutes" (example).

3. Click **Deactivate** (or **Stop using this role**, or **...** → **Deactivate**).

4. Confirm: "Deactivate Global Administrator?"

5. Click **Deactivate**.

**Result:** Your admin access is deactivated immediately (not waiting for the 1-hour timer). You're no longer Global Admin.

---

## The Scenario Check

Contoso's activation flow in action:

**Monday, 2:00 PM:** Jordan Kim needs to reset a departing employee's password.
- Jordan: "I need Global Admin access"
- Jordan navigates to PIM → **My roles** → **Global Administrator** → **Activate**
- Jordan selects: **Duration: 1 hour**, **Reason: "Password reset for departing employee"**
- Jordan submits the request

**Monday, 2:02 PM:** River Patel is notified (email: "Jordan Kim requested Global Admin activation").
- River opens PIM → **Approvals**
- River sees Jordan's request with duration and reason
- River clicks **Approve**
- River is notified: "Approved. Jordan will now complete activation."

**Monday, 2:03 PM:** Jordan sees the approval and completes activation.
- Jordan: "Activation is approved. Let me complete MFA."
- Jordan clicks **Activate**, completes MFA on her phone
- Jordan: "I'm now Global Admin for 1 hour (until 3:03 PM)"

**Monday, 2:05 PM:** Jordan is now admin and resets the password.
- Jordan navigates to **Users**, finds the departing employee, resets their password
- Action succeeds because Jordan is Global Admin

**Monday, 2:30 PM:** Jordan is done and deactivates early.
- Jordan navigates to PIM → **My roles** → **Active roles**
- Jordan clicks **Deactivate**
- Jordan: "I'm no longer admin. Risk window is closed."

**Result:** Admin access was granted for exactly the time needed (30 minutes), with approval and audit trail. No standing admin. Low risk.

---

## Check Your Understanding

- [ ] Describe the steps to request role activation in PIM
- [ ] Explain why MFA is required after approval but before activation
- [ ] State what happens to active role access when the timer expires
- [ ] Explain why an approver might deny an off-hours activation request
- [ ] Describe the scenario where an admin would manually deactivate before the timer expires

---

## Common Mistakes

**Forgetting MFA step.**
You request activation, it's approved, you try to use admin functions immediately. You get an error: "You must complete MFA to activate." Many users forget this step is coming. Make sure to have your phone ready before requesting activation.

**Requesting too-short duration.**
You request 30 minutes of Global Admin access. You start your admin task. 20 minutes in, you're mid-task when access expires. Now you must request again and wait for approval. Estimate how long tasks take: password resets (15–30 min), user provisioning (30–60 min), group creation (10–20 min).

**Leaving role active too long.**
You activate Global Admin for 2 hours but only need 20 minutes. You forget to deactivate. Access stays active for 1 hour 40 minutes longer than needed, increasing risk. Always manually deactivate when done.

**Submitting vague reasons.**
You request Global Admin activation with reason "Testing." The approver denies it (suspicious). Re-request with reason "Password reset for user ID #1234 (offboarding)." Be specific — it helps approvers and creates a better audit trail.

**Not following approval timelines.**
You request activation at 5 PM on Friday. Approver is already out of office. Request sits unapproved until Monday. For urgent requests, request well in advance or identify an approver who's available.

---

## What's Next

Module 7.4 covers **PIM Approval Workflows** — designing multi-level approvals, escalation, and emergency access. Module 7.5 covers **Auditing Privileged Actions** — how to investigate what admins did while they had access.

---

> 📚 **Entra ID — Zero to Admin** · Unit 7: Privileged Access Management
>
> [← Module 7.2 — Configuring PIM](../7-2-pim-configuration/7-2-pim-configuration.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 7.4 — PIM Approval Workflows →](../7-4-pim-approval/7-4-pim-approval.md)
