# Lifecycle Management — Auto-Expire Access, Force Reviews, and Clean Up
*Access should be temporary. After X days, it expires unless renewed. This prevents access creep.*

> 🟡 **Intermediate** · Unit 8, Module 8.4

---

## What You Will Learn

- Configure access expiration and automatic cleanup
- Set up access review requirements
- Enable renewal workflows
- Handle access expiration notifications
- Audit access lifecycle events

---

## Why This Matters

*SC-300: Plan and automate identity governance — Implement access lifecycle management*

The biggest risk in access governance is **access creep** — users accumulate access as they change roles, and no one removes the old access. Lifecycle management solves this: access expires automatically, forcing a choice: renew (intentional) or lose access (cleanup). This is the self-cleaning aspect of entitlement management.

---

## 🔄 Access Lifecycle States

```
Requested
  ↓
Approved
  ↓
Assigned (Active)
  ↓
[After X days]
  ↓
Expiration Notice Sent (30 days before expiration)
  ↓
[User must renew or access expires]
  ├─ Renewed → Active again
  └─ Not renewed → Expired
        ↓
     Cleanup: User removed from groups, apps, sites
```

---

## Lab 8.4.1 — Configure Access Expiration

**Tools:** Entra admin center
**Prerequisite:** Module 8.3 complete; access packages configured

Goal: Set access to expire after 90 days.

1. Navigate to **Entitlement Management** → **Access packages** → **Finance Team Access**.

2. Click **Lifecycle** (or **Expiration settings**, **Lifetime**).

3. Configure:
   - **After X days:** 90 days (access lasts 90 days, then expires)
   - **When access expires:** Auto-remove (user is removed from Finance-All group, Finance SharePoint, etc.)

4. Click **Save**.

**Result:** Any user granted "Finance Team Access" will have it for exactly 90 days, then auto-removed.

---

## Lab 8.4.2 — Configure Renewal Workflows

**Tools:** Entra admin center
**Prerequisite:** Lab 8.4.1 complete

Goal: Allow users to renew access before it expires.

1. On the same **Lifecycle** page, configure:
   - **Allow renewal?** Yes
   - **Days before expiration to request renewal:** 30 days

2. This means:
   - Day 1: Alex gets "Finance Team Access" (expires on day 91)
   - Day 60: Alex doesn't see renewal option (still 30+ days away)
   - Day 61: Alex sees notification: "Your access expires in 30 days. Click here to renew."
   - Day 61-90: Alex can request renewal
   - Day 91: If Alex renewed → access is extended 90 more days
   - Day 91: If Alex didn't renew → access expires, she's removed from Finance groups

3. Configure renewal approvers:
   - **Who approves renewals?** Same as original approval (Manager of requestor)
   - Or different: **Specific users** (e.g., Finance Director decides on renewals)

4. Click **Save**.

**Result:** Users can renew access proactively before expiration.

---

## Lab 8.4.3 — Enable Access Review

**Tools:** Entra admin center
**Prerequisite:** Lab 8.4.2 complete

Goal: Require managers to periodically review access.

1. On the **Lifecycle** page, find **Access review** section.

2. Configure:
   - **Require access review?** Yes
   - **Review frequency:** Every 90 days (same as duration)
   - **Review criteria:** Manager of user

3. This means:
   - Day 1: Alex gets access (review due on day 91)
   - Day 91: System sends to River (manager): "Alex has had Finance Team Access for 90 days. Does she still need it?"
   - River must review and approve/deny continuation

4. Click **Save**.

**Result:** Users' access is reviewed regularly (not just auto-renewed).

---

## Lab 8.4.4 — Monitor Expiration and Renewal Activity

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.4.3 complete; let some time pass or simulate expiring access

Goal: Track access that's expiring, being renewed, or expired.

1. Navigate to **Entitlement Management** → **Access packages** → **Finance Team Access** → **Assignments** (or **Active assignments**).

2. You'll see a list of users with this package, including:
   - **User name**
   - **Assigned on:** When they got access
   - **Expires on:** When access expires (column: "Expiration date")
   - **Status:** Active, Expiring soon, Expired

3. Filter by:
   - **Status: Expiring soon** (30-day renewal window)
   - **Status: Expired** (access has been removed)

4. Example list:
   - Alex Lee — Expires in 25 days (renewal window, should see notification)
   - Morgan Chen — Expires in 5 days (urgent renewal)
   - Sam Taylor — Expired 10 days ago (access removed, no longer in Finance-All group)

---

## Lab 8.4.5 — Test Renewal Workflow

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.4.4 complete; you have an assignment expiring soon

Scenario: Alex's access is about to expire. She renews it.

### Step 1: Access Renewal Notification

1. As Alex, you receive a notification:
   - **Subject:** "Your Finance Team Access is expiring in 30 days"
   - **Message:** "Click here to renew your access"

2. Click the link or navigate to **My Access** → **Approved access** → **Finance Team Access**.

---

### Step 2: Request Renewal

1. On the Finance Team Access section, click **Request renewal** (or **Renew**).

2. Fill in:
   - **Duration:** Default is 90 days (same as original)
   - **Business justification:** "I'm still in Finance, need continued access"

3. Click **Submit**.

**Result:** Renewal request is submitted to approver (your manager).

---

### Step 3: Approve Renewal (As Manager)

1. As River (manager), you see renewal request:
   - **User:** Alex Lee
   - **Package:** Finance Team Access
   - **Type:** Renewal
   - **Current expiration:** Day 91
   - **Requested duration:** 90 days (new expiration: Day 181)

2. Review:
   - Does Alex still need Finance access? (Check if she's still in Finance)
   - Yes → Click **Approve**
   - No → Click **Deny** (access will expire as originally planned)

3. If approved:
   - Alex's access is extended 90 more days
   - New expiration: Day 181
   - Process repeats: notification at Day 151

---

## Lab 8.4.6 — Handle Expired Access

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.4.5 complete; you have expired access

Goal: Understand and cleanup expired access.

1. Navigate to **Entitlement Management** → **Access packages** → **Finance Team Access** → **Assignments**.

2. Find **Status: Expired** users.

3. For each expired user:
   - **System automatically removed:**
     - Finance-All group membership ✓
     - Finance SharePoint access ✓
     - Finance app assignment ✓
   - **User can no longer access** Finance systems
   - **User can request again** if needed

4. Example:
   - Sam Taylor had access for 90 days
   - Did not renew
   - Day 91: System removed Sam from Finance-All, SharePoint, app
   - Sam: "I no longer have access"
   - Sam can request again if rehired or transferred back

---

## Lab 8.4.7 — Create Escalation for Expiration

**Tools:** Entitlement Management
**Prerequisite:** Lab 8.4.6 complete

Goal: Handle cases where users need extended access beyond normal duration.

**Scenario:** Alex needs Finance access for more than 90 days (special project). Set up exception.

1. On **Finance Team Access** package:
   - Default duration: 90 days
   - But some users need: 180 days (6 months)

2. Solution: Create an exception or higher-duration option:

   **Option A: Create a separate package**
   - "Finance Team Access (Long-term)" — 180 days, director approval
   - Use when you know user needs longer

   **Option B: Manual extension**
   - User's access expires
   - Before expiration, admin can grant "extended duration" exception
   - Request goes to director for approval

3. For now, use Option A:
   - Create "Finance Team Access (6-month)" package
   - Same resources
   - 180-day duration
   - Available to users with special projects

---

## The Scenario Check

**Contoso's access lifecycle:**

**Alex's access timeline:**
- Day 1: Alex requests "Finance Team Access" (90-day duration)
- Day 1: River approves
- Day 2: Access granted (Alex is in Finance-All group, Finance SharePoint, Finance app)
- Day 60: Notification sent: "Your access expires in 30 days"
- Day 65: Alex sees renewal option in My Access
- Day 70: Alex clicks "Renew" (90 more days)
- Day 70: Request sent to River for renewal review
- Day 71: River approves (Alex is still in Finance)
- Day 72: Access extended (new expiration: day 161)
- Day 131: New notification (day 161 is 30 days away)
- Day 135: Alex renews again
- Process repeats indefinitely (as long as she renews)

**Sam's access timeline (contractor):**
- Day 1: Sam requests "Finance Team Access" (90 days for Q2 project)
- Day 1: River approves
- Day 2: Access granted
- Day 88: Notification: "Access expires in 2 days"
- Day 88: Sam doesn't renew (project is ending)
- Day 91: System auto-removes Sam from Finance groups and app
- Sam: "I no longer have access" (correct, Q2 is over)

**Result:** Access is automatically managed. No manual cleanup needed. Expired access is instantly removed.

---

## Check Your Understanding

- [ ] Explain the difference between access expiration and access review
- [ ] State what happens when a user's access expires without renewal
- [ ] Describe why renewal notifications should come 30 days before expiration
- [ ] Explain how lifecycle management prevents "access creep"
- [ ] Design an expiration policy for a contractor vs. a full-time employee

---

## Common Mistakes

**Setting expiration too long.**
Access expires after 1 year (too long between reviews). Users forget they have access. Stale access accumulates. Set expiration to 90 days (quarterly review).

**Not enabling renewal.**
Access expires, user loses it mid-project. User is frustrated. Enable renewal: let active users keep access with manager approval. Or: auto-renew if no change needed (but still require periodic review).

**Expiration without notifications.**
Access expires. User doesn't know. User loses access suddenly. System should send notifications: 30 days before, 7 days before, 1 day before, and "access expired" notification.

**Not enforcing access review.**
Access expires and is auto-renewed without human review. Stale access stays. Require review: manager must affirmatively approve renewal (not auto-renew).

**Deleting assignment immediately on expiration.**
User's access expires. System instantly removes them from all groups. User's ongoing work breaks. Instead: soft removal (disable first, then hard delete after 30 days) OR: warn user before hard removal.

---

## What's Next

Module 8.5 is the **Governance Breakfix Lab** — a comprehensive lab where you design access packages, configure approval workflows, handle expirations, and manage a complete entitlement system.

---

> 📚 **Entra ID — Zero to Admin** · Unit 8: Identity Governance
>
> [← Module 8.3 — Approval Workflows](../8-3-approval-workflows/8-3-approval-workflows.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 8.5 — Breakfix: Governance Lab →](../8-5-governance-breakfix/8-5-governance-breakfix.md)
