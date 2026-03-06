# Breakfix: Identity Governance Lab
*Design and implement a complete entitlement system, then handle the full access lifecycle.*

> 🔴 **Advanced** · Unit 8, Module 8.5

---

## What You Will Learn

- Design a complete entitlement strategy for an organization
- Implement multiple access packages with different risk levels
- Configure approval workflows
- Test the full request-approval-grant-renew-expire cycle
- Audit and optimize governance

---

## The Scenario: Contoso's Governance Overhaul

**Company:** Contoso Ltd (50 people, 5 teams: Finance, Sales, Engineering, HR, Leadership)
**Current state:** Manual access (email requests, admins grant by hand, no expiration)
**Problem:**
- Alex left Finance 6 months ago, still has Finance access
- Morgan requested access once, no one ever checked if it's still needed
- No audit trail: "Who approved this access and when?"
- Compliance audit failed: "Prove you govern access"

**Your goal:** Implement entitlement management with request-approval-lifecycle workflows.

**Timeline:** 90 minutes to design, implement, test, and audit.

---

## Phase 1: Design Governance Strategy (30 minutes)

### Task 1.1 — Design Access Packages by Team

For each team, design one or two packages:

| Team | Package | Resources | Duration | Risk Level | Approver |
|---|---|---|---|---|---|
| **Finance** | Finance Team Access | Finance-All group, Finance SharePoint, Finance app | 90 days | Medium | Manager |
| **Finance** | Finance Manager Access | Finance-Managers group, Reports app, Analytics dashboard | 180 days | High | Director |
| **Sales** | Sales Team Access | Sales-All group, Salesforce, CRM app | 90 days | Medium | Manager |
| **Sales** | Sales Director Access | Sales-Directors group, Forecasting tool | 90 days | High | VP Sales |
| **Engineering** | Developer Access | Dev-All group, GitHub, Azure, Dev tools | 180 days | Medium | Tech Lead |
| **HR** | HR Team Access | HR-All group, HR systems, Employee database | 90 days | High | HR Director |
| **Leadership** | Executive Access | All groups, all apps, Executive dashboard | 365 days | Critical | CEO/CFO |

---

### Task 1.2 — Define Approval Rules

Create approval matrix:

| Package | Risk | Approver | SLA | Multi-stage? |
|---|---|---|---|---|
| Team Access | Medium | Manager | 24 hours | No |
| Manager Access | High | Manager + Director | 24 hours each | Yes |
| Executive Access | Critical | CEO | 24 hours | No (but logged) |
| Contractor Access | Medium | Manager | 12 hours | No |

---

### Task 1.3 — Define Lifecycle Rules

| Package | Duration | Renewal? | Review? | Auto-remove? |
|---|---|---|---|---|
| Team Access | 90 days | Yes (every 90 days) | Yes (manager reviews) | Yes |
| Manager Access | 180 days | Yes | Yes | Yes |
| Executive Access | 365 days | Yearly | Yes (CEO reviews) | Yes |
| Contractor Access | 90 days | No (one-time) | No | Yes |

---

## Phase 2: Implement Packages and Workflows (40 minutes)

### Task 2.1 — Create Finance Catalog and Packages

1. Create catalog: **Finance Catalog**

2. Create **Finance Team Access**:
   - Resources: Finance-All group, Finance SharePoint, Finance app
   - Duration: 90 days, renewal enabled
   - Approver: Manager of requestor
   - Requesters: All members

3. Create **Finance Manager Access**:
   - Resources: Finance-Managers group, Reports app, Analytics
   - Duration: 180 days, renewal enabled
   - Approver: Finance Director (specific user)
   - Requesters: Finance-Managers group only
   - Multi-stage: Manager → Director

---

### Task 2.2 — Create Sales and Engineering Packages

**Sales Catalog:**
- Sales Team Access (90 days, manager approval)
- Sales Director Access (90 days, VP Sales approval)

**Engineering Catalog:**
- Developer Access (180 days, tech lead approval)

---

### Task 2.3 — Create Contractor Package

Create a special package for temporary contractors:

**Contractor Temporary Access**
- Resources: (varies by project)
- Duration: 90 days, NO renewal (one-time)
- Approver: Hiring Manager
- Requesters: Specific users (HR, department heads)
- Auto-cleanup: Day 91, access removed

---

### Task 2.4 — Configure Approval Workflows

For each package, set approval configuration:

**Finance Team Access:**
- Approver: Manager of requestor (auto-route)
- SLA: 24 hours
- Escalation: If no response, escalate to Finance Director

**Finance Manager Access:**
- Stage 1: Manager of requestor (24 hours)
- Stage 2: Finance Director (24 hours)
- Both must approve
- Escalation: If no response, escalate to next level

---

### Task 2.5 — Configure Lifecycle Settings

For each package:
- Expiration: Set duration (90, 180, 365 days)
- Renewal: Enable (30-day window before expiration)
- Review: Enable (require approver to review every cycle)
- Notifications: 30 days before, 7 days before, 1 day before

---

## Phase 3: Test Request-Approval-Grant Cycle (30 minutes)

### Task 3.1 — Test Manager-Approved Package

**Scenario:** Alex (new Finance hire) requests "Finance Team Access."

1. As Alex, navigate to **My Access** → **Request access**.

2. Select **Finance Team Access**.

3. Fill in:
   - **Duration:** 90 days
   - **Reason:** "Hired as Finance Analyst, starting 3/1"

4. Submit request.

---

**As Manager (River):**

1. Navigate to **Entitlement Management** → **Approvals**.

2. See Alex's request.

3. Review:
   - Is Alex supposed to be in Finance? (Check HR data)
   - Reason is clear? (Yes, new hire)

4. Click **Approve** → "Alex is new Finance hire, approved."

---

**System Auto-Grants:**
- Adds Alex to Finance-All group
- Adds Alex to Finance SharePoint
- Assigns Finance app to Alex
- Sets expiration: Day 91

---

### Task 3.2 — Test Multi-Level Approval

**Scenario:** Morgan (Finance manager) requests "Finance Manager Access."

1. As Morgan, request "Finance Manager Access".
   - Reason: "Promoted to Finance Manager"

2. Request goes to Morgan's manager (River) → Approves.

3. Then goes to Finance Director (Alex Lee) → Approves.

4. System grants: Finance-Managers group, Reports app, Analytics (180-day duration).

---

### Task 3.3 — Test Denial Scenario

**Scenario:** Someone requests executive access but shouldn't have it.

1. Request "Executive Access".
   - Reason: "Want to see how executive dashboard works"

2. CEO (approver) reviews.

3. CEO denies: "Executive access not appropriate for non-leadership roles."

4. Requestor is notified and cannot access the package.

---

## Phase 4: Handle Lifecycle Events (15 minutes)

### Task 4.1 — Test Expiration Notification

**Scenario:** Alex's access is about to expire.

1. After some time (simulate: access expires in 30 days).

2. Alex receives notification:
   - **Subject:** "Your Finance Team Access expires in 30 days"
   - **Action:** Request renewal in My Access

---

### Task 4.2 — Test Renewal

1. As Alex, navigate to **My Access** → **Approved access**.

2. Find **Finance Team Access** (shows expiration date).

3. Click **Request renewal**.

4. Fill in:
   - **Duration:** 90 days (same as original)
   - **Reason:** "Still in Finance role, need continued access"

5. Submit renewal request.

---

**As Manager (River):**

1. Review renewal:
   - Alex still in Finance? (Check)
   - Approve → "Renewal approved, access extended 90 days"

2. Access is extended to Day 181.

---

### Task 4.3 — Test Expiration (No Renewal)

**Scenario:** Contractor's access expires without renewal.

1. Sam (contractor) has "Contractor Temporary Access" (90 days, no renewal).

2. Day 88: Sam receives "access expires in 2 days" notification.

3. Day 88: Sam does NOT request renewal (contract is ending).

4. Day 91: System auto-removes Sam from all groups/apps.

5. Sam: "I no longer have access" (correct).

---

## Phase 5: Audit and Optimize (15 minutes)

### Task 5.1 — Check Assignment Status

1. Navigate to **Entitlement Management** → **Access packages** → **Finance Team Access** → **Assignments**.

2. Review assignments:
   - Alex Lee — Expires in 55 days (active)
   - Morgan Chen — Expires in 175 days (active)
   - Sam Taylor — Expired 5 days ago (removed from groups)

---

### Task 5.2 — Audit Approval Activity

1. Navigate to **Entitlement Management** → **Approvals** (or **Audit logs**).

2. View history:
   - Alex's initial request → Approved by River
   - Morgan's request → Approved by River, then Alex
   - Contractor denied by manager
   - Sam's access expired and removed

---

### Task 5.3 — Generate Governance Report

1. Create a summary:

```
ENTITLEMENT MANAGEMENT SUMMARY (March 2024)

Access Packages:
- Finance Team Access: 15 active, 2 pending approval, 0 expired
- Finance Manager Access: 3 active, 0 pending, 0 expired
- Sales Team Access: 8 active, 1 pending, 1 expired (not renewed)
- Developer Access: 12 active, 2 pending, 0 expired
- Executive Access: 5 active, 0 pending, 0 expired

Approval Metrics:
- Total requests: 38
- Approved: 36 (95%)
- Denied: 2 (5%)
- Average approval time: 3 hours
- SLA compliance: 100% (all approved within 24-hour SLA)

Lifecycle Events:
- New assignments: 38
- Renewals: 5
- Expirations: 1 (expired without renewal, correctly cleaned up)
- Auto-removals: 1

Compliance Status:
✓ All access is approved
✓ All assignments are audited
✓ Access is time-limited and auto-expires
✓ Renewal process enforces review
✓ Complete audit trail exists
```

---

### Task 5.4 — Identify Optimization Opportunities

1. **Approval time is good** (3 hours, within 24-hour SLA) ✓

2. **Denial rate is good** (5%, catches suspicious requests) ✓

3. **Renewal process works** (users renew, managers review) ✓

4. **Possible improvements:**
   - Add more approvers (if queue ever backs up)
   - Monitor if renewal notifications are being read
   - Track which packages are most requested (optimize for those)

---

## Phase 6: Document Governance

**Create user guide:**

```
CONTOSO ENTITLEMENT MANAGEMENT GUIDE

How to Request Access:
1. Go to https://myaccess.microsoft.com
2. Click "Request access"
3. Select the package you need (Finance Team Access, Developer Access, etc.)
4. Provide business justification
5. Click "Request"
6. Wait for approver email (usually within 24 hours)

Common Packages:
- Finance Team Access: 90 days, auto-expires, renewable
- Developer Access: 180 days, auto-expires, renewable
- Contractor Access: 90 days, does NOT renew (one-time)

What Happens Next:
- Request is auto-routed to your manager (or specific approver)
- Manager reviews and approves/denies (usually same-day)
- If approved: You get access within 1 hour
- If denied: You're notified with reason

Renewing Access:
- 30 days before expiration: You'll see renewal option in My Access
- Click "Request renewal"
- Manager reviews again
- If approved: Access extended for another 90 days
- If denied: Access expires as planned

Questions?
Contact Jordan Kim (IT): jordan.kim@contoso.com
```

---

## The Scenario Check

**Contoso's Governance in Practice:**

**Week 1:** New Finance analyst (Alex) is hired. She requests "Finance Team Access." River approves within 1 hour. Alex gets access to Finance-All group, Finance SharePoint, and Finance app. Access is set to expire in 90 days.

**Week 4:** Morgan (Finance manager) requests "Finance Manager Access." River approves at level 1. Alex Lee (Finance Director) approves at level 2. Morgan gets manager-level access. Approval took 6 hours (both levels).

**Week 8:** Contractor Sam Taylor is brought on for a project. Sam requests "Contractor Temporary Access." Manager approves. Sam gets access for 90 days, no renewal option.

**Week 12 (Alex's 12-week mark):** System sends notification: "Your access expires in 30 days. Renew?"

**Week 12:** Alex renews. River approves. Access extended 90 more days.

**Week 12 (Sam's 12-week mark):** System sends notification: "Your access expires in 2 days." Sam doesn't renew (contract ending).

**Week 13:** Sam's access auto-expires. Removed from all Finance groups and app.

**Month 4:** Compliance audit arrives. Jordan (IT) provides report:
- 38 access requests, all approved with audit trail ✓
- All access is time-limited ✓
- All expired access is auto-removed ✓
- All approvals documented ✓

**Result:** Governance is now automatic, auditable, and compliant.

---

## Check Your Understanding

- [ ] Design an access package for your organization (resources, duration, approver)
- [ ] Explain the difference between time-limited and permanent access
- [ ] Describe the renewal process from user and approver perspectives
- [ ] State what happens to access after expiration
- [ ] Create a governance report for a compliance audit

---

## Common Mistakes

**Not testing before rolling out.**
You design packages perfectly but don't test approvals, renewals, or expirations. Users start requesting. Workflows break. Approvals never happen. Always test end-to-end before going live.

**Too many packages.**
You create "Finance Team Access," "Finance Basic Access," "Finance Shared Drive Access," etc. Users are confused which one to request. Consolidate: 1-2 packages per team max.

**Not communicating with users.**
You enable entitlement management and expect users to figure it out. Admins get flooded with "how do I request access?" Support users: send guide, hold training, monitor questions.

**Ignoring approval queue.**
Requests accumulate. Approvers don't see notifications. Requests sit for weeks. Monitor your queue daily. If backlog forms, escalate or add approvers.

**Not optimizing based on real usage.**
Finance Team Access gets 20 requests/day. System bogs down. Developer Access gets 1 request/month. Over-provisioned. Monitor metrics and adjust SLAs, approvers, or scope.

---

## What's Next

Unit 8 (Identity Governance) is now complete. You've automated access requests, designed approval workflows, and implemented self-cleaning access through expiration.

**Unit 9 (Devices and Compliance)** focuses on device management and compliance. You'll learn to register devices, enforce compliance policies, and manage device access to company resources.

---

> 📚 **Entra ID — Zero to Admin** · Unit 8: Identity Governance
>
> [← Module 8.4 — Lifecycle Management](../8-4-lifecycle-management/8-4-lifecycle-management.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Unit 9: Devices and Compliance →](../../UNIT%209%20-%20DEVICES%20AND%20COMPLIANCE/9-1-device-management-overview/9-1-device-management-overview.md)
