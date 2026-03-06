# Breakfix: PIM Governance Lab
*Design and implement a complete PIM strategy for a 50-person company, then test it end-to-end.*

> 🔴 **Advanced** · Unit 7, Module 7.6 · ⏱️ 120 minutes

---

## What You Will Learn

- Design a PIM governance strategy from scratch
- Configure roles, eligibility, approval workflows
- Test activation flow end-to-end
- Audit and verify compliance
- Troubleshoot and optimize PIM

---

## Why This Matters

*SC-300: Plan and automate identity governance — Design and implement PIM governance for an organization*

This lab brings together all PIM modules (7.1–7.5). Instead of learning individual concepts, you design a *complete strategy* for a company. Real-world PIM isn't just about enabling features — it's about balancing security, usability, compliance, and risk. This lab teaches you to make those trade-offs.

---

## The Scenario: Contoso Ltd Needs PIM

**Company:** Contoso Ltd (professional services, 50 employees)
**Current state:** No PIM. 8 Global Admins with permanent access.
**Problem:**
- Too many permanent admins (security risk)
- No approval for admin access (compliance risk)
- Admins can do anything, anytime (audit trail is weak)
- No emergency access procedure (operational risk)

**Your goal:** Design and implement PIM governance for Contoso.

**Timeline:** 120 minutes to complete the entire strategy.

---

## Phase 1: Design PIM Governance (30 minutes)

### Task 1.1 — Identify Admin Roles

Contoso's current admins and their responsibilities:

| Admin | Primary role | Secondary role | Frequency |
|---|---|---|---|
| Jordan Kim | IT, password resets | User management | Daily (5+ times/day) |
| River Patel | HR, onboarding/offboarding | — | 2–3 times/week |
| Alex Lee | Finance, billing admins | — | 1–2 times/week |
| Morgan Chen | Engineering, service principal setup | Application admin | 2–3 times/week |
| Sam Taylor | Security, compliance | Conditional Access | 1–2 times/week |

---

### Task 1.2 — Classify Roles by Risk

For each admin role in Entra ID, classify by risk:

| Admin Role | Risk Level | Typical Users | Frequency |
|---|---|---|---|
| **Global Administrator** | CRITICAL | Should be minimal | Rare |
| **Security Administrator** | HIGH | Security team | Rare |
| **Application Administrator** | MEDIUM | Dev/app teams | Moderate |
| **User Administrator** | MEDIUM | HR, IT | Frequent |
| **Cloud Application Administrator** | MEDIUM | App owners | Moderate |

---

### Task 1.3 — Design PIM Strategy

Complete the design matrix:

| Role | Permanent or Eligible? | Approver | Approval Required? | Max Duration | Require MFA? | Notifications? |
|---|---|---|---|---|---|---|
| Global Admin | Eligible | River + Alex | Yes (2 approver) | 1 hour | Yes | Yes |
| Security Admin | Eligible | River | Yes | 2 hours | Yes | Yes |
| User Admin | Eligible | None | No | 4 hours | Yes | No |
| Application Admin | Eligible | Alex | No | 2 hours | Yes | No |
| Cloud App Admin | Eligible | None | No | 2 hours | Yes | No |

**Rationale:**
- **Global Admin:** Highest risk → highest scrutiny (2 approvers, 1 hour max, MFA required)
- **Security Admin:** High risk → moderate scrutiny (1 approver, 2 hours)
- **User Admin:** Medium risk, frequent use → lighter scrutiny (no approval, 4 hours allowed)
- **Application Admin:** Medium risk → light scrutiny (no approval)

---

### Task 1.4 — Plan Approval Workflow

**Approval process for Global Admin activation:**

```
User requests activation
       ↓
Check: Is it business hours? Is duration ≤ 1 hour?
       ├─ No → Alert admins (off-hours/long request)
       └─ Yes → Continue
       ↓
Route to Primary Approver (River)
       ↓
If River approves in < 1 hour: Done ✓
If River doesn't respond in 1 hour: Escalate to Alex
       ↓
Alex approves or denies
       ↓
If approved: Activation ready for user
If denied: User notified, investigation if needed
```

---

### Task 1.5 — Plan Emergency Access

**Emergency procedure for critical incidents:**

- Break-glass account: `admin_emergency@contoso.onmicrosoft.com`
- Break-glass role: Global Administrator (permanent, no PIM required)
- Usage: Only during critical incidents (downtime > 15 minutes, all approvers unreachable)
- Post-incident: Audit the emergency account, document usage, rotate credentials

---

## Phase 2: Implement PIM Configuration (45 minutes)

### Task 2.1 — Convert Permanent Admins to Eligible

**For each current permanent Global Admin:**

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Global Administrator**.

2. In **Eligible assignments**, add them as eligible:
   - Click **+ Add assignment**
   - Select the admin
   - Set **Assignment type: Eligible**
   - Click **Assign**

3. In **Active assignments**, remove them as permanent:
   - Find them in the Active list
   - Click **...** → **Remove**
   - Confirm

4. **Repeat for other roles:**
   - Security Administrator: Convert to eligible
   - User Administrator: Convert to eligible
   - Etc.

**Result:** No permanent admins (except emergency break-glass).

---

### Task 2.2 — Configure Role Settings

**For Global Administrator:**

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles** → **Global Administrator** → **Settings**.

2. Configure:
   - **Require approval to activate:** Yes
   - **Select approvers:** River Patel, Alex Lee
   - **Require Azure MFA:** Yes
   - **Max activation duration:** 1 hour
   - **Send notifications:** Yes

3. Click **Save**.

**For User Administrator:**

1. Open **User Administrator** → **Settings**.

2. Configure:
   - **Require approval:** No (frequent role, lighter restrictions)
   - **Require Azure MFA:** Yes
   - **Max activation duration:** 4 hours
   - **Send notifications:** Yes

3. Click **Save**.

**For Application Administrator:**

1. Open **Application Administrator** → **Settings**.

2. Configure:
   - **Require approval:** No
   - **Require Azure MFA:** Yes
   - **Max activation duration:** 2 hours
   - **Send notifications:** Yes

3. Click **Save**.

---

### Task 2.3 — Set Approval Rules and SLAs

1. Navigate to **Privileged Identity Management** → **Settings** (tenant-wide).

2. Configure:
   - **Approval SLA for Global Admin:** 30 minutes (must be approved within 30 min)
   - **Alert if off-hours:** Yes (alert if Global Admin activated after 6 PM)
   - **Alert if multiple activations:** Yes (alert if same user activates > 3 times per day)
   - **Require reason for activation:** Yes

3. Click **Save**.

---

### Task 2.4 — Set Up Emergency Access (Break-Glass)

1. Create a test break-glass account:
   - **Name:** `admin_emergency`
   - **Role:** Make this account a **permanent** Global Administrator (exception to the rule)
   - **Reason:** Emergency-only account for incidents

2. Configure PIM for this account:
   - **Skip PIM requirements:** If possible (emergency account should have immediate access)
   - **Or:** Set minimal approval (auto-approve)

3. Secure this account:
   - **Enable MFA:** Enforce (require phone/authenticator)
   - **Create strong password:** Store in secure location (password vault, printed, physically locked)
   - **Restrict access:** Use CA policy to restrict sign-in locations (office only, or block unless emergency)

4. **Document:**
   - How to access break-glass account
   - When to use it (critical incidents only)
   - Post-incident procedures

---

## Phase 3: Test Activation Flow (30 minutes)

### Task 3.1 — Request Global Admin Activation (As Jordan)

Scenario: Jordan (IT admin) needs to reset a user's password. She must request Global Admin activation.

1. Sign in as **jordan.kim@yourdomain.onmicrosoft.com** (InPrivate browser).

2. Navigate to **Privileged Identity Management** → **My roles** → **Eligible roles**.

3. Find **Global Administrator** (should be eligible, not active).

4. Click **Activate**.

5. Fill out:
   - **Duration:** 1 hour (max allowed)
   - **Reason:** "Password reset for departing employee Alex Lee"
   - Click **Activate**

6. **Expected result:** Request is submitted. You see "Request submitted."

---

### Task 3.2 — Approve as River Patel

Scenario: River (primary approver) receives the activation request.

1. Sign in as **river.patel@yourdomain.onmicrosoft.com** (different InPrivate window).

2. Navigate to **Privileged Identity Management** → **Approvals** (or **Pending approvals**).

3. You should see Jordan's request:
   - **User:** Jordan Kim
   - **Role:** Global Administrator
   - **Duration:** 1 hour
   - **Reason:** "Password reset for departing employee Alex Lee"

4. Review the request:
   - Is this legitimate? (Yes, password resets are normal)
   - Is the reason clear? (Yes, specific task)
   - Is the duration reasonable? (Yes, 1 hour for password reset is appropriate)

5. Click **Approve**.

6. **Add comment (optional):** "Approved. Legitimate password reset."

7. Click **Approve** (confirm).

8. **Expected result:** Approval is complete. River gets confirmation.

---

### Task 3.3 — Activate (As Jordan)

Back as Jordan:

1. In **Privileged Identity Management** → **My roles**, see that your request is now "Approved."

2. Click the role (or click **Activate**).

3. **Complete MFA:**
   - Use your phone/authenticator to verify
   - Entra ID sends a notification or code
   - Approve or enter code

4. Click **Activate** (confirm).

5. **Expected result:** You see "Global Administrator is now active. Expires in 59 minutes."

---

### Task 3.4 — Perform Admin Task (As Jordan)

Now that you're activated:

1. Navigate to **Users** → **All users**.

2. Find a test user (example: **alex.lee**).

3. Click on them → **Reset password**.

4. **Set new password:** (Entra ID generates temporary password)

5. Click **Reset**.

6. **Expected result:** Password is reset. This action succeeds because you're now Global Admin.

---

### Task 3.5 — Check Activation Duration

1. In **Privileged Identity Management** → **My roles** → **Active roles**.

2. You should see **Global Administrator** with a countdown timer:
   - "Expires in 45 minutes" (if 15 minutes have passed)
   - Timer counts down in real-time

---

### Task 3.6 — Deactivate (As Jordan)

Once you're done:

1. In **Privileged Identity Management** → **My roles** → **Active roles**.

2. Click **Deactivate** (or **Stop using this role**).

3. Confirm.

4. **Expected result:** You are no longer Global Admin. Timer is gone.

---

## Phase 4: Audit and Compliance (15 minutes)

### Task 4.1 — Check PIM Activity Logs

1. Navigate to **Privileged Identity Management** → **Activity** (or **Audit logs**).

2. You should see your activation:
   - **User:** Jordan Kim
   - **Action:** Requested Global Administrator activation
   - **Status:** Approved
   - **Approver:** River Patel
   - **Time:** [timestamp]

3. Review the full lifecycle:
   - Request → Approval → Activation → Deactivation
   - All events logged ✓

---

### Task 4.2 — Correlate with Azure AD Audit Logs

1. Navigate to **Audit logs** (main, not PIM).

2. Filter:
   - **User:** Jordan Kim
   - **Date:** Today

3. You should see:
   - Password reset activity (the action you performed while Global Admin)
   - Time stamp matches your activation window

4. **Verification:** Admin actions are logged and correlate with PIM activation ✓

---

### Task 4.3 — Generate Compliance Report

1. Navigate to **Privileged Identity Management** → **Reports** (or export **Activity** logs).

2. Filter:
   - **Date range:** Last 24 hours
   - **Role:** Global Administrator
   - **Status:** All

3. Export or generate:
   - You should see the activation(s) from today
   - Includes approver, duration, user actions during activation

4. **Save report:** This is your compliance documentation.

---

## Phase 5: Troubleshooting and Optimization (10 minutes)

### Task 5.1 — Test Denial Scenario

Scenario: Someone requests Global Admin activation but it's denied.

1. Request activation (as a test user) with:
   - **Duration:** 8 hours (too long!)
   - **Reason:** "Testing" (vague!)
   - **Time:** 10 PM (off-hours!)

2. As River (approver), review the request and click **Deny**.

3. Provide reason: "Request is off-hours and request duration exceeds maximum. Resubmit during business hours with specific purpose."

4. As the requester, see the denial and understand why.

5. **Learning:** Denials provide feedback. Submitting better requests next time.

---

### Task 5.2 — Document PIM Governance

Create a one-page document for Contoso admins:

```
CONTOSO PIM POLICY

Who needs PIM approval?
- Global Administrator (any activation)
- Security Administrator (any activation)

Who doesn't need approval?
- User Administrator
- Application Administrator
- Cloud Application Administrator

How to request activation:
1. Go to Privileged Identity Management → My roles
2. Click "Activate" on the role you need
3. Select duration (max allowed shown)
4. Provide reason (required)
5. Click "Activate"
6. Wait for approval (30 min SLA)
7. Complete MFA
8. You're now admin for [duration]

Who approves?
- Global Admin & Security Admin: River Patel, Alex Lee
- Others: See Settings

What if I need access after hours?
- Emergency access: Contact on-call admin
- Or: Request via PIM with reason "Emergency" (will be reviewed)

What if I'm done early?
- Go to My roles → Active roles → Deactivate
- Reduces risk window

Questions?
- Email IT: jordan.kim@contoso.com
```

---

### Task 5.3 — Optimize Based on Usage

After a week of PIM usage, check:

1. **Average approval time:** Should be < 30 min
   - If > 30 min: Add more approvers or reduce approval requirements

2. **Denial rate:** Should be 5–20% (some suspicious requests)
   - If > 50%: Approval rules too strict
   - If < 2%: Approval rules too loose (not catching suspicious requests)

3. **User feedback:** Do admins complain about friction?
   - If yes: Consider auto-approval for lower-risk roles
   - If no: Current settings working well

4. **Make adjustments:**
   - Reduce approval time for User Admin (no approval, it's fast)
   - Increase max duration if User Admin is regularly hitting 4-hour limit
   - Add approvers if approval backlog is building

---

## The Scenario Check: Contoso's First Week with PIM

**Day 1 (Monday):**
- Convert all 8 Global Admins from permanent to eligible
- Configure role settings per design
- Notify admins of PIM launch
- Jordan (first admin) requests Global Admin access to reset password
- River approves in 5 minutes
- Jordan activates (completes MFA)
- Jordan resets password
- Deactivates
- Activity logged ✓

**Day 2–5 (Tuesday–Friday):**
- Several activations per day (password resets, user provisioning, group creation)
- All approvals flow smoothly
- Average approval time: 12 minutes
- One denial: Request was off-hours, resubmitted next business day
- No friction reported (good balance)

**End of Week:**
- Check audit logs: 24 activations, 23 approved, 1 denied
- Check correlations: All activations had corresponding admin actions
- Generate compliance report: "Contoso is now PIM-governed"
- Auditors approve: "PIM is properly configured and used"

**Result:** Security improved, compliance improved, admin productivity maintained.

---

## Check Your Understanding

- [ ] Design a PIM strategy for a 20-person organization
- [ ] Explain trade-offs between high security (strict approvals) and high usability (fast access)
- [ ] Describe how to audit whether admins are following PIM requirements
- [ ] State what you would do if a user requests Global Admin access after-hours with vague reason
- [ ] Create an emergency access procedure for your tenant

---

## Common Mistakes

**Overcomplicating PIM on launch.**
You set up 15 approval rules, require 3 approvers for everything, set max durations to 30 minutes. Admins hate PIM. Workarounds emerge. Users stop using PIM and try other methods. Start simple: approve high-risk roles, auto-approve low-risk. Iterate based on feedback.

**Not communicating PIM to users.**
You enable PIM without telling admins. Admins try to use their roles as usual. They're confused when they suddenly need approval. Support tickets pile up. Document PIM *before* launch. Train admins. Provide a how-to guide.

**Forgetting to convert all permanent admins.**
You make some admins eligible but leave others as permanent. The permanent admins become a security gap. Someone can still use those accounts without going through PIM. Convert everyone (except emergency break-glass).

**Setting unreasonable approval SLAs.**
You require approval within 5 minutes. Approvers can't respond that fast. Requests timeout. Admins are blocked. Set realistic SLAs: 30 minutes for business hours is reasonable. 4 hours for after-hours (lower priority).

**Not optimizing based on real usage.**
You launch PIM with a generic config. After 3 weeks, you see: User Admin activations are common (10+ per day), but all auto-approve (no approval), so friction is low. Separately, Global Admin requests from non-admins are constantly denied. You don't adjust. You could improve by: raising max duration for User Admin, or understanding why non-admins are requesting Global Admin. Review metrics and iterate.

---

## What's Next

Unit 7 (Privileged Access Management) is now complete. You've learned to control admin access through PIM, ensuring admins have access only when needed, for as long as needed, with oversight.

**Unit 8 (Identity Governance)** shifts focus from *privilege control* to *access governance*. You'll learn to automate access requests, set up approval workflows for app access, manage entitlements, and enforce access reviews company-wide.

---

> 📚 **Entra ID — Zero to Admin** · Unit 7: Privileged Access Management
>
> [← Module 7.5 — Auditing Privileged Actions](../7-5-pim-auditing/7-5-pim-auditing.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Unit 8: Identity Governance →](../../UNIT%208%20-%20IDENTITY%20GOVERNANCE/8-1-entitlement-management-overview/8-1-entitlement-management-overview.md)
