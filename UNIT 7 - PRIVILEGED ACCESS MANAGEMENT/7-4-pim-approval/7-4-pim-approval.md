# PIM Approval Workflows
*Design approval processes that balance security with speed, and handle emergencies.*

> 🟡 **Intermediate** · Unit 7, Module 7.4

---

## What You Will Learn

- Understand different approval models (single approver, multi-level, self-approval)
- Configure approval chains with primary and escalation approvers
- Set up emergency access procedures (break-glass accounts)
- Design approval rules based on risk (high-privilege roles, unusual requests)
- Monitor approval queue and SLAs

---

## Why This Matters

*SC-300: Plan and automate identity governance — Design PIM approval workflows for compliance and security*

Module 7.3 showed how to request and approve activation. This module shows how to *design approval processes*. Bad approval design creates bottlenecks (approval queue backs up) or security gaps (approvers are absent, requests bypass approval). Good design ensures approvals are fast, secure, and resilient.

---

## 🔀 Three Approval Models

### Model 1: Single Approver (Fast but Risky)

**Design:** One person can approve all activations.

| Aspect | Detail |
|---|---|
| **Approver** | River Patel (only person who can approve Global Admin requests) |
| **Speed** | Fast (if River is available) |
| **Risk** | If River is on vacation, requests queue up unapproved |
| **Compliance** | Weak (one person is bottleneck) |
| **Use case** | Small teams, low-risk roles |

**Problem:** Single point of failure. If River is out sick, no one can approve, and admins can't work.

---

### Model 2: Multi-Approver (Balanced)

**Design:** Multiple people can approve. Any one of them can approve.

| Aspect | Detail |
|---|---|
| **Approvers** | River Patel, Alex Lee, Jordan Kim (all admins) |
| **Speed** | Medium (one of three can approve) |
| **Risk** | Reduced (if River is out, Alex/Jordan can approve) |
| **Compliance** | Good (multiple approvers, audit trail) |
| **Use case** | Most enterprises |

**Advantage:** Resilience. Requests don't queue up just because one person is out.

---

### Model 3: Escalation Chain (Complex but Secure)

**Design:** Primary approver approves if available. If absent, escalate to secondary approver.

| Aspect | Detail |
|---|---|
| **Primary approver** | River Patel (first to approve) |
| **Secondary approver** | Alex Lee (if River unavailable for 2 hours) |
| **Tertiary approver** | Jordan Kim (if both River and Alex unavailable) |
| **Speed** | Slow (escalation takes time) |
| **Risk** | Very low (requires escalation, more scrutiny) |
| **Compliance** | Very good (clear chain, all requests eventually approved/denied) |
| **Use case** | Highly regulated industries, multi-level approval requirement |

---

## 📋 Approval Rules Based on Risk

Different requests should have different approval rules:

| Request Scenario | Risk Level | Approval Rule |
|---|---|---|
| Global Admin, business hours, 1 hour | Medium | Any approver can approve |
| Global Admin, after 6 PM, 8 hours | High | Multiple approvers required (2 of 3) |
| Global Admin, from foreign IP, any time | Critical | Manager approval + Security approval (two roles) |
| User Admin, business hours, 4 hours | Low | Auto-approve (no approval required) |
| Exchange Admin, any time | Medium | Escalation chain (primary approver) |

---

## Lab 7.4.1 — Design an Approval Chain

**Tools:** Entra admin center
**Prerequisite:** Module 7.2 complete

Goal: Configure a multi-level approval workflow.

### Step 1: Define Approval Hierarchy

For Contoso, define:

**Primary approver:** River Patel
- First person to be asked when someone requests Global Admin activation

**Secondary approver:** Alex Lee
- If River doesn't respond in 2 hours, escalate to Alex

**Tertiary approver:** Jordan Kim
- If both River and Alex don't respond in 4 hours, escalate to Jordan

---

### Step 2: Configure in PIM

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles** → **Global Administrator** → **Settings**.

2. Find **Approvers** section.

3. Add multiple approvers:
   - Click **+ Add approvers** (or **+ Add**)
   - Select **River Patel**
   - Click **Select**

4. Add secondary:
   - Click **+ Add secondary approver** (if available)
   - Select **Alex Lee**
   - Set escalation time: **2 hours** (if River doesn't respond in 2 hours, escalate)

5. Add tertiary:
   - Click **+ Add tertiary approver** (if available)
   - Select **Jordan Kim**
   - Set escalation time: **4 hours**

6. Click **Save**.

---

### Step 3: Verify Configuration

1. The **Approvers** section should now show:
   - Primary: River Patel
   - Secondary: Alex Lee (if no response in 2 hours)
   - Tertiary: Jordan Kim (if no response in 4 hours)

---

## Lab 7.4.2 — Configure Risk-Based Approval Rules

**Tools:** Entra admin center
**Prerequisite:** Lab 7.4.1 complete

Advanced: Set approval rules based on risk factors (time of day, duration, location).

### Step 1: Access Advanced Approval Settings

1. Navigate to **Privileged Identity Management** → **Settings** (tenant-wide settings).

2. Look for **Approval settings** or **Advanced approval**.

---

### Step 2: Set Approval Rules

Configure rules like:

**Rule 1: Off-hours activation requires escalation**
- **Condition:** Activation requested between 6 PM and 9 AM (after hours)
- **Action:** Require secondary approver (escalation to Alex)
- **Rationale:** Off-hours requests are higher risk

**Rule 2: Long-duration activation requires multiple approvers**
- **Condition:** Activation duration > 4 hours (long session)
- **Action:** Require approval from two different approvers
- **Rationale:** Longer admin access = higher risk

**Rule 3: Unusual user activation**
- **Condition:** User doesn't typically activate roles (first-time activation)
- **Action:** Require manager approval (not just any approver)
- **Rationale:** Unusual activity might indicate account compromise

---

### Step 3: Verify Rules

After configuration, test:

1. Request Global Admin activation at **3 AM** (off-hours)
   - Should require escalation/secondary approver

2. Request Global Admin for **8 hours** (long duration)
   - Should require multiple approvers

3. Request as a user who rarely activates
   - Should require manager approval

---

## Lab 7.4.3 — Configure Emergency Access (Break-Glass)

**Tools:** Entra admin center
**Prerequisite:** Module 4.1 (break-glass accounts configured)

Goal: Set up an emergency approval bypass for critical situations.

**Scenario:** All approvers are unreachable (incident at 2 AM, all admins out of office). You need Global Admin access immediately to respond to an incident. What happens?

### Step 1: Create Emergency Role Access

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles** → **Global Administrator** → **Settings**.

2. Find **Emergency access** or **Break-glass approval** section.

3. Configure:
   - **Allow emergency activation:** Yes (allow bypass if approvers unreachable)
   - **Duration:** 1 hour (minimal time for emergency)
   - **Requires MFA:** Yes (still require verification even in emergency)
   - **Notification:** Yes (alert admins that emergency access was used)

4. Click **Save**.

---

### Step 2: Document Emergency Procedure

Create a documented emergency procedure:

```
EMERGENCY GLOBAL ADMIN ACCESS PROCEDURE

When to use:
- Critical incident affecting tenant availability
- All regular approvers unreachable
- Incident requires immediate Global Admin access

How to activate:
1. In PIM, request "Global Administrator" activation
2. Duration: Select "Emergency" or 1 hour (minimum)
3. Reason: Describe incident (for audit trail)
4. If approvers don't respond in 5 minutes, use emergency activation
5. Complete MFA
6. You are now Global Admin for 1 hour

After incident:
1. Document what you did (for investigation)
2. Notify approvers (email incident summary)
3. Deactivate early when incident is resolved
4. Security team reviews the emergency activation log
```

---

## Lab 7.4.4 — Monitor Approval Queue and SLAs

**Tools:** Entra admin center
**Prerequisite:** Lab 7.4.3 complete; you have some approval requests (real or test)

Goal: Ensure approvals don't back up.

### Step 1: Check Pending Approvals

1. As an approver (River), navigate to **Privileged Identity Management** → **Approvals** (or **Pending approvals**).

2. You'll see a list of requests awaiting your approval:
   - User requesting activation
   - Role requested
   - Duration
   - Time submitted
   - Status: "Pending your approval"

3. For each request, note:
   - How long it's been pending
   - Is it within SLA? (example: Should be approved within 4 hours)

---

### Step 2: Set Up Approval SLAs

SLA = Service Level Agreement. Example: "Approve all activation requests within 4 hours."

**Configure SLA:**

1. In **Privileged Identity Management** → **Settings**, find **Approval SLA** or **Approval deadline**.

2. Set rules like:
   - **Global Admin requests:** Approve within 4 hours
   - **User Admin requests:** Approve within 8 hours (lower risk)
   - If not approved by deadline: Auto-deny or escalate to secondary approver

3. Click **Save**.

---

### Step 3: Check Approval Metrics

1. Navigate to **Reports** or **Compliance reports** (if available in your PIM).

2. Look for metrics like:
   - **Average approval time:** How long does approval take?
   - **Approval rate:** What % of requests are approved vs. denied?
   - **Overdue approvals:** Requests past SLA deadline?

3. **Goals:**
   - Average approval time: < 30 minutes (for business hours)
   - Approval rate: 80–90% approved (20–10% denied for suspicious requests)
   - Overdue: 0 (all requests handled within SLA)

---

## Lab 7.4.5 — Test Approval Workflow (End-to-End)

**Tools:** Entra admin center
**Prerequisite:** Labs 7.4.1-7.4.4 complete

Full end-to-end test of your approval workflow.

### Step 1: Request Activation (Unusual Scenario)

As Jordan (an admin), request Global Admin activation **after hours** (simulate 8 PM):

1. In PIM → **My roles** → **Eligible roles** → **Global Administrator** → **Activate**

2. Fill in:
   - **Duration:** 4 hours (longer than usual)
   - **Reason:** "Emergency user migration" (legitimate-sounding but off-hours)
   - **Time of request:** 8 PM

3. Submit.

---

### Step 2: Review as Primary Approver (River)

As River, check approvals:

1. Navigate to **Privileged Identity Management** → **Approvals**.

2. See Jordan's request:
   - **User:** Jordan Kim
   - **Role:** Global Administrator
   - **Duration:** 4 hours (longer than usual — elevated risk)
   - **Time:** 8 PM (after hours — elevated risk)
   - **Reason:** "Emergency user migration"

3. Evaluate:
   - Is this legitimate? (migration is legitimate, but 4 hours for migration?)
   - Should I approve or deny? (Risks: off-hours, long duration. Reasons: emergency migration is real)

---

### Step 3: Approve with Comment

1. Click **Approve**.

2. Add comment: "Approved. Off-hours emergency migration justified. Duration 4 hours is acceptable."

3. Click **Approve** (confirm).

---

### Step 4: Activate (As Jordan)

Back as Jordan:

1. In PIM → **My roles**, see the approved request.

2. Click **Activate**.

3. Complete MFA.

4. **Global Admin access is active for 4 hours.**

---

### Step 5: Audit the Request

1. As an admin/auditor, navigate to **Privileged Identity Management** → **Activity** (or **Audit logs**).

2. Find the request:
   - Time submitted: 8 PM
   - Requested by: Jordan Kim
   - Role: Global Administrator
   - Approver: River Patel
   - Status: Approved
   - Activation time: 8:05 PM
   - Expiration: 12:05 AM (4 hours later)

3. This full audit trail is now recorded for compliance/investigation.

---

## The Scenario Check

Contoso's approval workflow:

**Role Activations at Contoso:**

1. **Monday 10 AM:** Jordan requests Global Admin for 1 hour (business hours, normal request)
   - River approves immediately (within 5 minutes)
   - Jordan activates and completes task
   - Approval time: 5 minutes ✓

2. **Tuesday 7 PM:** Alex requests Global Admin for 2 hours (after hours, elevated risk)
   - Approval rule: Off-hours requests → escalate to secondary approver
   - Jordan (secondary) is paged, approves within 15 minutes
   - Alex activates
   - Approval time: 15 minutes ✓

3. **Wednesday 2 AM:** Critical incident. Morgan requests Global Admin. All approvers unreachable.
   - Morgan uses emergency activation (configured in Lab 7.4.3)
   - No approval needed (emergency override)
   - Morgan activates and mitigates incident
   - Alert sent: "Emergency Global Admin activation used"
   - Post-incident: Security team reviews what Morgan did (all audited)
   - Approval time: Immediate (emergency) ✓

**Result:** Approvals are fast, resilient, and audited. No requests get stuck in queue. Emergencies have a path. All activity is logged.

---

## Check Your Understanding

- [ ] Explain the difference between single-approver and multi-approver workflows
- [ ] Describe when you would use escalation (primary/secondary approvers)
- [ ] State what "emergency activation" means and when it's used
- [ ] Explain why off-hours requests might have different approval rules
- [ ] Design an approval workflow for a 50-person organization with 3 admins

---

## Common Mistakes

**Configuring approval but not monitoring SLA.**
You set up approvals. Requests start coming in. Approvers don't respond within SLA. Requests back up. Two weeks later, admins are frustrated: "I requested activation 10 days ago and still haven't been approved." Monitor SLA compliance. If requests are consistently delayed, escalate, or add more approvers.

**Making approval too strict.**
You configure: Global Admin requires approval from 3 different roles (admin, security, manager). A user requests activation. All three must independently approve. Takes 4 hours. Most requests die in queue. Admins stop requesting and try to work around the system. Be realistic: 1–2 approvers for high-risk roles, auto-approve for low-risk.

**Not providing emergency access path.**
You set up strict approval for Global Admin. Critical incident at 2 AM. Approvers are unreachable. Admin can't activate. Incident spirals out of control. Always provide an emergency override (with audit logging). Incident response is more important than normal approval rules.

**Approver fatigue.**
You configure approvers for 20 different roles. Every approval goes to the same people. They're overwhelmed. Requests back up. Approvers get angry and approve everything without reading. Distribute approval responsibilities: different people approve different roles based on expertise.

**Not reviewing approval metrics.**
You set up PIM and forget about it. Six months later, you audit and find: average approval time is 8 hours (users waiting too long). Many critical requests were denied (too strict). Requests are piling up (no one is monitoring). Review metrics monthly: approval times, approval rates, SLA compliance.

---

## What's Next

Module 7.5 covers **Auditing Privileged Actions** — how to log and investigate what admins do while they have access. Module 7.6 is a hands-on lab where you design a full PIM governance strategy.

---

> 📚 **Entra ID — Zero to Admin** · Unit 7: Privileged Access Management
>
> [← Module 7.3 — Using PIM (Just-in-Time Access)](../7-3-pim-activation/7-3-pim-activation.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 7.5 — Auditing Privileged Actions →](../7-5-pim-auditing/7-5-pim-auditing.md)
