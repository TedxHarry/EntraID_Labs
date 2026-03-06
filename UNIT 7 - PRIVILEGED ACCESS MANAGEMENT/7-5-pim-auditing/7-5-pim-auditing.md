# Auditing Privileged Actions
*Log every admin action. Review logs to ensure admins are doing what they should, not what they shouldn't.*

> 🟡 **Intermediate** · Unit 7, Module 7.5

---

## What You Will Learn

- Understand what PIM logs and what it doesn't
- Access PIM audit logs and sign-in logs for admins
- Investigate suspicious admin activity
- Generate compliance reports from PIM audit data
- Detect privilege escalation and abuse

---

## Why This Matters

*SC-300: Plan and automate identity governance — Audit privileged access and investigate anomalies*

PIM controls *when* admins can access roles and *for how long*. Auditing verifies *what they did* while they had access. Together, they create accountability: admins know their actions are logged, which discourages abuse. In incidents, auditing lets you understand the scope of an attacker's access.

---

## 📋 What Gets Logged

### PIM Activation Logs
**What's logged when:**
- User requests activation
- Admin approves/denies request
- User completes MFA and activates
- Role is deactivated (manual or expiration)

**Example entry:**
- **Time:** 2024-03-05 2:30 PM
- **User:** Jordan Kim
- **Action:** Requested Global Administrator activation
- **Duration:** 1 hour
- **Reason:** "Password reset for departing employee"
- **Approver:** River Patel
- **Status:** Approved

### Azure AD Audit Logs
**What's logged when an active admin:**
- Creates/deletes users
- Modifies group memberships
- Resets passwords
- Creates app registrations
- Modifies Conditional Access policies
- Grants app permissions
- Changes tenant settings

**Example entry:**
- **Time:** 2024-03-05 2:35 PM
- **User:** Jordan Kim (admin)
- **Activity:** Reset user password
- **Target:** alex.lee@contoso.com
- **Result:** Success
- **IP address:** 10.0.0.5 (corporate office)
- **Device:** Windows laptop

### Sign-In Logs for Admins
**What's logged when an admin signs in:**
- Time of sign-in
- Location (IP, city, country)
- Device type (Windows, mobile, web browser)
- Sign-in success/failure
- MFA result (success, bypassed, denied)

---

## 🔍 Audit Log Analysis

### Scenario 1: Investigate a Large User Deletion

**Alert:** 50 users were deleted on 3/5 at 3 PM.

**Investigation steps:**

1. **Check PIM logs:** Who was activated as Global Admin around 3 PM?
   - Find: Jordan Kim activated Global Admin at 2:30 PM (1-hour duration, until 3:30 PM)
   - Matches the deletion timeframe ✓

2. **Check the deletion details:**
   - Who deleted the users? Jordan Kim ✓
   - When? 3:05 PM – 3:15 PM (within her activation window) ✓
   - How many? 50 users ✓
   - Reason? Audit log shows no reason (admin just deleted without context)

3. **Ask Jordan:** "Why did you delete 50 users?"
   - Legitimate answer: "They were test accounts for the new department, onboarding was cancelled"
   - Suspicious answer: "I didn't delete anyone" (attacker impersonated her)

4. **Conclusion:** If legitimate, document and move on. If suspicious, initiate incident response (Module 6.5).

---

### Scenario 2: Detect Unauthorized Permission Grant

**Alert:** An Exchange Online admin granted a suspicious app `Mail.ReadWrite.All` permission.

**Investigation:**

1. **Check PIM logs:** Who activated Exchange Online Admin?
   - Find: River Patel activated at 9:00 AM (2-hour duration)
   - Activity in question: 9:15 AM (within her activation window)

2. **Check audit logs:** What permission was granted?
   - **App:** "Data Export Tool" (unrecognized app)
   - **Permission:** `Mail.ReadWrite.All` (read and modify all mailboxes)
   - **Granted by:** River Patel (confirmed from PIM logs)
   - **Time:** 9:15 AM

3. **Ask River:** "Did you grant permission to 'Data Export Tool'?"
   - Legitimate: "Yes, for the data migration project"
   - Suspicious: "No, I didn't do that" (attacker compromised River's account)

4. **If suspicious:** This is credential compromise or account takeover. Initiate incident response.

---

## Lab 7.5.1 — Access PIM Activity Logs

**Tools:** Entra admin center
**Prerequisite:** Module 7.4 complete; you have some activation history

Goal: Find PIM activation logs.

### Step 1: Navigate to Activity Logs

1. Navigate to **Privileged Identity Management** → **Activity** (or **Audit**, or **History**).

2. You'll see a list of events, typically showing:
   - **Date/time**
   - **User**
   - **Action** (Activated, Requested, Approved, Deactivated, etc.)
   - **Role**
   - **Status** (Success, Denied, Expired, etc.)

---

### Step 2: Review Activation History

1. Expand your own recent activations (if you have any).

2. For each activation, note:
   - **Request time:** When you requested
   - **Approval time:** When it was approved
   - **Activation time:** When you activated
   - **Expiration:** When it expired
   - **Duration:** How long you had access

3. This is the **PIM activation lifecycle**.

---

### Step 3: Filter Activity Logs

1. Use filters to narrow down:
   - **User:** Filter by specific admin (example: "Jordan Kim")
   - **Action:** Filter by activation, request, approval, deny
   - **Role:** Filter by role (example: "Global Administrator")
   - **Date range:** Last 7 days, last 30 days, custom

2. Example query: "Show all Global Administrator activations in the last week"
   - Result: A list of every time someone activated that role

3. This is useful for:
   - Compliance audits: "How many times was Global Admin activated?"
   - Incident investigation: "Who was Global Admin at 3 PM on 3/5?"
   - Trend analysis: "Is Global Admin being overused?"

---

## Lab 7.5.2 — Correlate PIM Logs with Azure AD Audit Logs

**Tools:** Entra admin center
**Prerequisite:** Lab 7.5.1 complete; you have activation history

Goal: Connect PIM activation with actual admin actions.

### Step 1: Find a PIM Activation

1. In **Privileged Identity Management** → **Activity**, find a recent activation.
   - **Example:** Jordan Kim activated Global Administrator at 2:30 PM, for 1 hour

---

### Step 2: Note the Activation Window

- **Start time:** 2:30 PM
- **End time:** 3:30 PM
- **Role:** Global Administrator
- **User:** Jordan Kim

---

### Step 3: Check Azure AD Audit Logs During That Window

1. Navigate to **Audit logs** (or **Monitoring** → **Audit logs**).

2. Filter:
   - **User:** Jordan Kim
   - **Date range:** 2:30 PM – 3:30 PM (the activation window)

3. You'll see a log of what Jordan did while admin:
   - Example log entries:
     - 2:35 PM: Reset password for alex.lee@contoso.com
     - 2:40 PM: Added user to "Finance-All" group
     - 2:45 PM: Created new app registration "DataSync"
     - 3:00 PM: Granted "DataSync" app permission `User.ReadWrite.All`

---

### Step 4: Investigate Actions

1. For each action, ask: "Is this legitimate?"
   - Password reset: ✓ Legitimate (user was leaving, password reset is normal)
   - Adding to group: ✓ Legitimate (user was transferred to Finance, needs Finance access)
   - Creating app: ? Suspicious (why create an app during this activation?)
   - Granting permission: ? Suspicious (why grant `User.ReadWrite.All` to a new app?)

2. **Conclusion:** Most actions are legitimate, but the app creation and broad permission grant warrant investigation:
   - Ask Jordan: "Why did you create 'DataSync' app and grant it `User.ReadWrite.All`?"
   - If legitimate: "We were setting up a sync service for the Finance team"
   - If suspicious: This could be an attacker creating persistence

---

## Lab 7.5.3 — Generate Compliance Report

**Tools:** Entra admin center
**Prerequisite:** Lab 7.5.2 complete

Goal: Create a report showing PIM compliance for auditors.

### Step 1: Access Reporting

1. Navigate to **Privileged Identity Management** → **Reports** (or **Compliance**, or **Audit reports**).

2. Available reports might include:
   - **Audit trail:** All PIM activities (requests, approvals, activations)
   - **Activation history:** When roles were activated, duration, by whom
   - **Approval decisions:** Requests approved vs. denied

---

### Step 2: Generate Activation Report

1. Select **Activation history** report.

2. Configure:
   - **Date range:** Last 3 months (for quarterly compliance)
   - **Role:** All roles (or specific, like Global Administrator)
   - **Status:** All (Approved, Denied, Expired, Deactivated)

3. Click **Generate** (or **Export**).

4. The report shows:
   - **Total activations:** How many times roles were activated
   - **Total duration:** Sum of all activation time (example: 40 hours total for Global Admin in Q1)
   - **Approvers:** Who approved each activation
   - **Denials:** How many requests were denied (and why, if recorded)

---

### Step 3: Use Report for Compliance

1. A compliance auditor asks: "Prove you're using PIM and roles are controlled."

2. You provide this report, showing:
   - Global Administrator was activated 8 times in 3 months (not overused ✓)
   - Each activation required approval ✓
   - Average duration: 2 hours (reasonable ✓)
   - No denials (requests are generally legitimate ✓)
   - All actions are logged and auditable ✓

3. Auditor is satisfied: "PIM is properly configured and used."

---

## Lab 7.5.4 — Detect Suspicious Admin Patterns

**Tools:** Entra admin center, analysis
**Prerequisite:** Lab 7.5.3 complete

Goal: Identify admin activity that doesn't look right.

### Pattern 1: High-Frequency Activations

**Red flag:** One admin activates Global Administrator 20 times per day.

**Investigation:**
- Is this normal? (No, typically admins activate a few times per week)
- Possibility: Script or automated task (needs PIM activations repeatedly)
- Possibility: Attacker with stolen credentials, testing access

**Action:** Check the activations:
- Are they all during business hours? (Normal)
- Are they from different IP addresses? (Suspicious)
- Do they correlate with actual admin tasks in audit logs? (Should correlate)

---

### Pattern 2: Off-Hours Activation

**Red flag:** Global Administrator is activated at 3 AM.

**Investigation:**
- Is this normal? (Depends on org — might be normal for 24/7 ops)
- Reason stated: What reason did the admin provide?
- Approver response: Did approver approve without question?

**Action:**
- If reason is "Emergency on-call incident," legitimate ✓
- If reason is blank or vague ("Testing"), investigate

---

### Pattern 3: Unusually Long Duration

**Red flag:** Admin requests Global Administrator for 8 hours (maximum allowed).

**Investigation:**
- Is this task expected to take 8 hours?
- Did the admin use the full 8 hours, or deactivate early?
- If used the full 8 hours every time, might be trying to minimize activation overhead

**Action:**
- Talk to admin: "Why do you need 8-hour activations?"
- Adjust: If 2-hour tasks, set max to 2 hours to reduce risk window

---

### Lab 7.5.5 — Audit Admin Use of Sensitive Operations

**Tools:** Entra admin center
**Prerequisite:** Lab 7.5.4 complete

Goal: Track sensitive admin actions (password resets, permission grants, etc.).

### Step 1: Create Alert for Sensitive Operations

1. Navigate to **Audit logs**.

2. Set up filters to track:
   - **User resets password:** Any admin resetting user passwords
   - **Group membership modified:** Any admin adding/removing users to admin groups
   - **Application permission granted:** Any admin granting app permissions
   - **Conditional Access policy changed:** Any admin modifying security policies

---

### Step 2: Monitor for Abuse

1. Example: Admin resets password for a non-existent user.
   - **Suspicious:** Attacker creating hidden account?

2. Example: Admin adds themselves to "Global Administrators" group.
   - **Suspicious:** Privilege escalation?
   - **Reality check:** Admin might already be Global Admin, adding themselves to group is redundant

3. Example: Admin grants random app `User.ReadWrite.All` permission without documentation.
   - **Suspicious:** Attacker creating persistence?

4. Example: Admin disables Conditional Access policies before performing actions.
   - **Suspicious:** Attacker disabling controls before malicious action?

---

### Step 3: Create Incident Response Trigger

If you detect suspicious patterns:

1. Immediately revoke the admin's role activation (force deactivate)
2. Reset their password (Module 6.5)
3. Review sign-in logs (Module 6.2)
4. Investigate audit logs (what did they do?)
5. Contact the admin (verify it's really them)

---

## The Scenario Check

Contoso's audit process:

**Weekly Audit (Jordan performs):**
1. Check PIM activity logs
   - 12 activations last week: 10 Global Admin, 2 User Admin
   - All were approved, all had reasons
   - Average duration: 1.5 hours
   - No unusual patterns

2. Correlate with Azure AD audit logs
   - Check what was actually done during each activation
   - Confirm activities match the stated reasons
   - No admin actions outside activation windows (good control)

3. Check for alerts
   - No off-hours activations
   - No unusually long durations
   - No permission grants to suspicious apps
   - Green light ✓

**Quarterly Audit (Jordan prepares for compliance):**
1. Generate PIM compliance report
   - 52 total activations in Q1
   - 48 approved, 4 denied (approval working ✓)
   - No role overuse (Global Admin 8 times only)
   - All with documented reasons

2. Present to auditor
   - "PIM is active and properly governed"
   - "All admin access is logged and audited"
   - "Approval workflows are enforced"
   - Auditor signs off: Compliance verified ✓

---

## Check Your Understanding

- [ ] List three things that PIM audit logs record
- [ ] Explain how to correlate a PIM activation with actual admin actions
- [ ] Describe what a suspicious pattern looks like (frequency, time, duration)
- [ ] State what you would do if an admin had an activation denied
- [ ] Design an audit strategy for detecting privilege escalation

---

## Common Mistakes

**Not reviewing audit logs regularly.**
You enable PIM, set up activations, then never check logs. Months later, an insider activates Global Admin and modifies all users. You discover it during compliance audit — too late. Review PIM and audit logs weekly (at minimum).

**Correlating only PIM logs, not actions.**
You check "Jordan activated Global Admin at 2 PM." You assume everything's fine. You don't check what Jordan actually did during that activation. Maybe Jordan deleted 100 users or granted a suspicious app broad permissions. Always correlate: activation → actual admin actions.

**Ignoring denied requests.**
A user requests Global Administrator. You deny it (suspicious request). The user never inquires why, never tries again. This might indicate an attacker (realized the request was denied, gave up). Investigate denials: reach out to the user, confirm they actually made the request, or flag for incident response.

**Not generating compliance reports.**
You have audit data, but you don't summarize it. When auditors ask, "Prove admins are controlled," you can't produce a clean report. Generate monthly or quarterly reports showing: total activations, approval rate, activation duration trends, any suspicious patterns. It's both good governance and good defense.

**Setting audit retention too short.**
You keep audit logs for 30 days (free tier). An incident is discovered 60 days later. Logs are gone. Can't investigate. Set longer retention (90 days minimum, longer if possible). In regulated industries, keep logs for 1–3 years.

---

## What's Next

Module 7.6 (Breakfix Lab) is a comprehensive lab where you design and test a full PIM governance strategy. You'll configure roles, approval workflows, test activations, and audit results.

---

> 📚 **Entra ID — Zero to Admin** · Unit 7: Privileged Access Management
>
> [← Module 7.4 — PIM Approval Workflows](../7-4-pim-approval/7-4-pim-approval.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 7.6 — Breakfix: PIM Governance Lab →](../7-6-pim-breakfix/7-6-pim-breakfix.md)
