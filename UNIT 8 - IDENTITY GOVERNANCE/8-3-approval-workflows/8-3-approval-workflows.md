# Approval Workflows — Control Who Approves Access Requests
*Route requests to managers, directors, or specific roles. Automate or require human judgment.*

> 🟡 **Intermediate** · Unit 8, Module 8.3

---

## What You Will Learn

- Configure approval settings for access packages
- Route requests to managers automatically
- Set up multi-level approvals (manager + director)
- Configure escalation for non-responsive approvers
- Monitor approval queue and SLAs

---

## Why This Matters

*SC-300: Plan and automate identity governance — Implement approval workflows for access requests*

An access package without approvals is a free-for-all. Approvals ensure that only legitimate requests are granted. Well-designed workflows route requests to the right decision-maker (usually the user's manager). This module teaches you to build approval workflows that are fast, secure, and scalable.

---

## 🔄 Approval Models

### Model 1: Manager Approval (Standard)
**Route to:** User's direct manager
**Automatic:** Yes, system auto-routes based on org chart
**Use case:** Most access requests

**Flow:**
- User requests access
- System checks: Who is this user's manager?
- Request auto-routes to manager
- Manager approves/denies
- Instant (fast)

---

### Model 2: Specific Approvers
**Route to:** Specific users or groups (hardcoded)
**Automatic:** No, you specify
**Use case:** Restricted packages, specialized access

**Example:**
- "Executive Access" package → Always route to CEO or CFO
- "Security Access" package → Always route to Security team

---

### Model 3: Multi-Level Approval
**Route to:** Manager, then Director
**Automatic:** Staged approval
**Use case:** High-risk, high-privilege packages

**Flow:**
- User requests "Finance Manager Access"
- Level 1: Finance Manager (immediate approver)
- If approved → Level 2: Finance Director (secondary approval)
- If both approve → Access granted
- If either denies → Request denied

---

### Model 4: Escalation (Non-Responsive Approver)
**Route to:** Manager, then escalate if no response
**Automatic:** Time-based
**Use case:** Ensure requests don't get stuck

**Example:**
- Request goes to manager
- If manager doesn't respond in 24 hours → escalate to director
- Director must respond within 24 hours
- Prevents bottlenecks

---

## Lab 8.3.1 — Configure Manager-Based Approval

**Tools:** Entra admin center
**Prerequisite:** Module 8.2 complete; "Finance Team Access" package exists

Goal: Set up automatic manager approval for Finance Team Access.

1. Navigate to **Entitlement Management** → **Access packages** → **Finance Team Access**.

2. Click **Approval settings** (or go to **Edit** → **Approval settings**).

3. Configure:
   - **Require approval?** Yes
   - **Who approves?** Select **Manager of requestor** (auto-route to user's manager)

4. Configure escalation:
   - **If manager doesn't respond in:** 14 days
   - **Escalate to:** Next manager level (user's manager's manager) OR Backup approvers

5. Click **Save**.

**Result:** When users request "Finance Team Access," the system automatically routes to their manager for approval.

---

## Lab 8.3.2 — Configure Multi-Level Approval

**Tools:** Entra admin center
**Prerequisite:** Lab 8.3.1 complete; "Finance Manager Access" package exists

Goal: Set up Finance Manager Access to require both manager and director approval.

1. Navigate to **Finance Manager Access** package.

2. Click **Approval settings**.

3. Configure Multi-stage approval:

   **Stage 1:**
   - Approver type: **Manager of requestor**
   - Reason: "Manager verifies that user is indeed a manager"

   **Stage 2:**
   - Approver type: **Specific users** → Select Finance Director (example: Alex Lee)
   - Reason: "Director approves high-privilege manager-level access"

4. Configure escalation:
   - Both stages: If no response in 7 days, escalate to next approver
   - Prevent bottlenecks

5. Click **Save**.

**Result:** "Finance Manager Access" requests go through two approval stages.

---

## Lab 8.3.3 — Test Approval Workflow

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.3.2 complete; approval settings configured

Scenario: Alex (Finance employee) requests "Finance Team Access." River (her manager) approves.

### Step 1: Request (As Alex)

1. Navigate to **My Access** → **Request access**.

2. Select **Finance Team Access**.

3. Fill in:
   - **Duration:** 90 days
   - **Business justification:** "I'm joining the Finance team"

4. Click **Submit**.

**Result:** Request is submitted. Notification is sent to River (Alex's manager).

---

### Step 2: Review Request (As Manager/River)

1. Navigate to **Entitlement Management** → **Approvals** (or open email notification).

2. You should see the pending request:
   - **Requestor:** Alex Lee
   - **Package:** Finance Team Access
   - **Justification:** "I'm joining the Finance team"
   - **Duration:** 90 days

3. Review:
   - Is Alex supposed to be in Finance? (Check with HR/department)
   - Does 90 days make sense? (Yes)
   - Is the justification clear? (Yes)

4. Click **Approve** (or **Deny** if suspicious).

5. **Add comments (optional):** "Verified: Alex transferred to Finance on 3/1. Access approved."

6. Click **Approve** (confirm).

**Result:** Request is approved. Alex is notified: "Your access has been approved."

---

### Step 3: Access Granted (Automatic)

1. After approval, the system automatically:
   - Adds Alex to Finance-All group
   - Adds Alex to Finance SharePoint site
   - Assigns Finance app to Alex
   - Sends notification: "Your access is active for 90 days"

2. Alex: "I have access!"

**Result:** Access is provisioned automatically (no manual steps needed).

---

## Lab 8.3.4 — Test Denial Scenario

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.3.3 complete

Scenario: Someone requests access but doesn't have legitimate need. Test denial.

### Step 1: Request (Suspicious)

1. Request "Finance Team Access" with:
   - **Justification:** "Curious about Finance" (vague/suspicious)

2. Submit.

---

### Step 2: Deny (As Manager)

1. As manager, see the request.

2. Review:
   - Justification is vague ("Curious" is not a business need)
   - No indication user is transferring to Finance
   - Decision: Deny

3. Click **Deny**.

4. **Add reason (required):** "No legitimate business justification. Finance access is restricted to team members only."

5. Click **Deny** (confirm).

**Result:** Request is denied. User is notified with the reason.

---

## Lab 8.3.5 — Monitor Approval Metrics

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.3.4 complete; you have approval activity

Goal: Check approval performance.

1. Navigate to **Entitlement Management** → **Reports** (if available) or **Approvals**.

2. Check metrics:
   - **Total requests:** How many access requests?
   - **Approved:** How many approved?
   - **Denied:** How many denied?
   - **Average approval time:** How long does approval take?
   - **Pending:** Any requests waiting?

3. Example healthy metrics:
   - Total: 20 requests/month
   - Approved: 18 (90%)
   - Denied: 2 (10% for suspicious requests)
   - Average approval time: 4 hours (fast)
   - Pending: 0 (no backlogs)

4. If metrics are bad:
   - Approval time > 1 day: Add more approvers or adjust SLAs
   - Denial rate > 30%: Approval rules too strict?
   - High pending: Approvers overloaded?

---

## Lab 8.3.6 — Configure SLA and Escalation

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.3.5 complete

Prevent approval bottlenecks with SLAs (Service Level Agreements).

1. Navigate to **Entitlement Management** → **Settings** (or package approval settings).

2. Configure:
   - **Approval SLA:** 24 hours (must be approved within 24 hours)
   - **If not approved by SLA:** Escalate to next approver

3. Example SLA policy:
   - Manager has 24 hours to respond
   - If no response: Escalate to department director
   - Director has 24 hours to respond
   - If still no response: Auto-deny (or auto-approve depending on risk)

4. Click **Save**.

**Result:** Requests won't sit in queue indefinitely.

---

## The Scenario Check

**Contoso's approval workflows:**

1. **Finance Team Access** (manager approval):
   - Request → Auto-route to user's manager
   - SLA: 24 hours
   - Escalation: If no response in 24 hours, escalate to Finance Director
   - Result: Average approval time = 2 hours

2. **Finance Manager Access** (multi-level approval):
   - Request → Manager (level 1) → Finance Director (level 2)
   - Both must approve
   - SLA: 24 hours per level
   - Result: Average approval time = 8 hours (two levels)

3. **Developer Access** (manager approval):
   - Request → Engineering Manager
   - Escalation: If no response in 24 hours → Tech Lead
   - SLA: 24 hours
   - Result: Average approval time = 3 hours

When Alex requested Finance Team Access, River (her manager) approved within 1 hour. When a Finance manager requested elevated "Finance Manager Access," River approved it, then Alex Lee (Finance Director) approved it. The complete approval process took 6 hours.

---

## Check Your Understanding

- [ ] Explain why manager-based approval is preferred (vs. fixed approvers)
- [ ] Describe what "escalation" means in approval workflows
- [ ] State the purpose of SLA in approvals
- [ ] Design a multi-level approval workflow for a sensitive package
- [ ] Explain what happens when an approval request is denied

---

## Common Mistakes

**Not setting SLA.**
Requests are submitted but never approved. Weeks pass. Users are frustrated. Approvers don't know they're behind. Set a 24-hour SLA: forces action.

**Escalating to overloaded people.**
Approval escalates to the CEO. CEO is overwhelmed. Requests pile up. Escalate to someone who can actually respond (department director, not CEO).

**Multi-level approval that's too strict.**
"Finance Manager Access" requires: manager + director + CFO. Three approvals take 3 days. By the time it's approved, the manager needed access 2 days ago. Limit multi-level to high-risk packages only.

**Not monitoring approval queue.**
Requests are submitted, approvers are sent notifications, but then... nothing. Notifications go to spam. Approvers forget. Monitor your approval queue weekly. If requests are pending > 48 hours, follow up with approvers.

**Auto-denying after SLA.**
You set SLA to 24 hours. If no response, auto-deny. But approver was on vacation. Request auto-denied without review. Either: escalate instead of auto-deny, or set SLAs realistically (48 hours accounting for vacations).

---

## What's Next

Module 8.4 covers **Lifecycle Management and Access Reviews** — how to handle access expiration, renewals, and periodic reviews. Module 8.5 is a breakfix lab.

---

> 📚 **Entra ID — Zero to Admin** · Unit 8: Identity Governance
>
> [← Module 8.2 — Access Packages](../8-2-access-packages/8-2-access-packages.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 8.4 — Lifecycle Management →](../8-4-lifecycle-management/8-4-lifecycle-management.md)
