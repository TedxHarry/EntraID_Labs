# Entitlement Management — Govern Who Gets Access and How
*Let users request the access they need. Let approvers control it. Automate the rest.*

> 🟡 **Intermediate** · Unit 8, Module 8.1 · ⏱️ 50 minutes

---

## What You Will Learn

- Understand what entitlement management is and why organizations need it
- Explain the concept of access packages and catalogs
- Describe how users request access
- Understand approval workflows for access requests
- Identify the difference between standing access and time-limited access

---

## Why This Matters

*SC-300: Plan and automate identity governance — Design and implement entitlement management*

Units 1–7 taught you to *control access* through Conditional Access, PIM, and security controls. Unit 8 teaches you to *govern access systematically* through a request-approval-expiration process. Instead of admins manually adding users to groups (slow, error-prone), entitlement management lets users request access, managers approve, and access auto-expires. This scales governance to large organizations.

---

## 🔑 Core Concepts

### Problem: Manual Access Management (Bad)

**Current state at a company:**
- Alex requests access to "Finance Shared Drive"
- Email goes to her manager, River
- River manually adds Alex to "Finance-All" group
- Alex gets access (days later)
- Alex leaves Finance, no one removes her
- One year later, Alex still has Finance access (access creep)

**Problems:**
- Slow (email, manual steps)
- Error-prone (River forgets to remove access)
- Not auditable (no record of why Alex got access)
- Not compliant (no approval workflow)

---

### Solution: Entitlement Management (Good)

**New state with entitlement management:**
- Alex navigates to access portal: "I need Finance access"
- She selects "Finance Shared Drive" access package
- She provides reason: "Transferred to Finance team"
- Request goes to River (auto-routed)
- River approves
- Access is auto-granted (Alex gets group membership)
- Access expires in 90 days (reviewed automatically)
- Notification: "Your access expires in 30 days. Will you still need it?"
- If not renewed, access is auto-revoked

**Benefits:**
- Fast (self-service, no email chains)
- Auditable (every request is logged)
- Compliant (approval trail exists)
- Self-cleaning (auto-expiration removes stale access)

---

## 📦 Key Components

### Access Packages
An **access package** is a bundle of resources (groups, apps, SharePoint sites) that a user might need.

**Examples:**
- "Finance Team Access" = Finance-All group + Finance SharePoint site + Excel shared folder access
- "Developer Access" = Dev-All group + GitHub organization + Azure subscription + Development apps (Slack, Jira)
- "Manager Access" = Managers group + HR systems + Budget app + Reports app
- "Sales Access" = Sales-All group + Salesforce + CRM + Customer database

---

### Catalogs
A **catalog** is a collection of access packages available for users to request.

**Example:**
- **Contoso Public Catalog:**
  - Finance Team Access (Finance, 90 days)
  - Sales Team Access (Sales, 90 days)
  - Developer Access (Engineering, 180 days)
  - Executive Access (Leadership, 1 year)

---

### Approval Workflows
**How access is approved:**

| Workflow Type | Approver | Use Case |
|---|---|---|
| **Manager-approved** | User's direct manager | Low-risk access (joining a team) |
| **Multi-approver** | Manager + Department head | Medium-risk access (privileged team) |
| **Automatic** | No approval needed | Low-risk, self-service access |
| **No approval** | User's manager, after 90 days | Review access, renew or revoke |

---

### Time-Limited Access
Access granted **for a period, then expires.**

**Examples:**
- Finance Team Access: 90 days (then expires, can renew)
- Contractor Access: 6 months (then revoked automatically)
- Temporary Project Access: 30 days (project-specific, short-term)

**Benefits:**
- Auto-cleanup (no stale access)
- Forced reviews (user must renew or lose access)
- Reduced risk (compromised account has time-limited damage)

---

## 🎯 End-to-End Flow

```
User needs access
       ↓
User browses access packages in My Access portal
       ↓
User selects "Finance Team Access"
       ↓
User provides reason: "Transferred to Finance"
       ↓
Request is submitted
       ↓
System auto-routes to manager for approval
       ↓
Manager receives notification:
  "Alex requested Finance Team Access (reason: transferred)"
       ↓
Manager approves (or denies)
       ↓
If approved: Alex is added to Finance-All group, Finance SharePoint, etc.
       ↓
Access is time-limited (90 days)
       ↓
After 60 days: Notification "Your Finance access expires in 30 days"
       ↓
Day 90: If not renewed, access expires (Alex is removed from groups)
       ↓
Alex can request again if needed
```

---

## Lab 8.1.1 — Understand Entitlement Management in Your Tenant

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 1.1 complete

> 🔑 **Licensing:** Entitlement Management requires **Entra ID P2**. Your dev tenant includes this.

1. Navigate to **Identity Governance** → **Entitlement Management** (or search "access packages").

2. You'll see sections:
   - **Catalogs** — Collections of access packages
   - **Access packages** — Individual bundles of resources
   - **Requests** — User requests for access
   - **Approvals** — Pending approval decisions
   - **Access reviews** — Reviews and expirations

3. For now, just familiarize yourself with the layout. You'll populate these in next modules.

---

## Lab 8.1.2 — Understand Access Lifecycle

**Time:** 15 minutes
**Tools:** Conceptual
**Prerequisite:** Lab 8.1.1 complete

Draw or describe the **access lifecycle** for an employee:

### Scenario: Alex Joins Finance Team

**Day 1: Request**
- Alex: "I've been transferred to Finance. I need access to Finance systems."
- Alex requests "Finance Team Access" package
- Status: Pending approval

**Day 1 (1 hour later): Approval**
- River (manager) sees the request
- River approves: "Yes, Alex is now part of Finance team"
- Status: Approved

**Day 2: Fulfillment**
- System auto-adds Alex to:
  - Finance-All security group
  - Finance-Shared SharePoint site
  - Finance reports app
- Alex: "I have access"
- Status: Active

**Day 30: Notification**
- System: "Your Finance access expires in 60 days. Do you still need it?"
- Alex: "Yes, I need to keep it"
- Alex clicks "Renew"
- Expiration extended another 90 days

**Day 60: Expiration (if not renewed)**
- If Alex didn't renew:
  - System auto-removes Alex from Finance-All group
  - Finance access is revoked
  - Status: Expired

**Benefits of this lifecycle:**
- Auditable: Every step is logged
- Self-cleaning: Access automatically expires
- Forced review: User must actively renew (not forgotten)
- Compliant: Approval trail is documented

---

## Lab 8.1.3 — Compare Manual vs. Entitlement-Managed Access

**Time:** 15 minutes
**Tools:** Comparison table
**Prerequisite:** Lab 8.1.2 complete

Create a comparison table for two scenarios:

### Scenario A: Manual Access (Old Way)

| Step | Owner | Time | Audit Trail |
|---|---|---|---|
| 1. Alex emails River: "I need Finance access" | Alex | Day 1 | Email in inbox |
| 2. River manually adds Alex to Finance-All group | River | Day 2 | Audit log (buried) |
| 3. Alex: "Thanks, I have access" | Alex | Day 2 | No record |
| 4. Months pass, Alex leaves Finance | Alex → new team | Month 4 | No notification |
| 5. No one removes access, Alex still in Finance-All | (bug) | Ongoing | Discovered during audit |
| 6. Access review: "Why is Alex still in Finance?" | Auditor | Month 12 | Confusing history |

**Problems:** Slow, manual, not auditable, access creep, compliance gaps.

---

### Scenario B: Entitlement-Managed Access (New Way)

| Step | Owner | Time | Audit Trail |
|---|---|---|---|
| 1. Alex requests "Finance Team Access" | Alex | Day 1, 10:05 AM | Request logged |
| 2. System routes to River | System | Day 1, 10:06 AM | Workflow log |
| 3. River approves | River | Day 1, 10:15 AM | Approval logged |
| 4. Access granted automatically | System | Day 1, 10:16 AM | Fulfillment logged |
| 5. Day 91: Notification "access expires" | System | Month 3 | Expiration notice sent |
| 6. No renewal, access auto-revoked | System | Month 3 | Revocation logged |
| 7. Access review: "Access was requested, approved, granted, reviewed, revoked" | Auditor | Anytime | Clear history ✓ |

**Benefits:** Fast, auditable, self-cleaning, compliant.

---

## Lab 8.1.4 — Design Entitlement Strategy for Contoso

**Time:** 10 minutes
**Tools:** Design document
**Prerequisite:** Lab 8.1.3 complete

Design entitlement management for Contoso Ltd (50 people, 5 teams):

**Step 1: Identify Access Packages**

| Team | Access Package | Resources | Duration |
|---|---|---|---|
| Finance | Finance Team Access | Finance-All group, Finance SharePoint, Budget app | 90 days |
| Sales | Sales Team Access | Sales-All group, Salesforce, CRM app | 90 days |
| Engineering | Developer Access | Dev-All group, GitHub, Azure, Dev tools | 180 days |
| HR | HR Access | HR-All group, HR systems, Employee database | 90 days |
| Leadership | Executive Access | Exec-All group, All apps, Analytics dashboard | 365 days |

**Step 2: Define Approvers**

- Finance Access → River Patel (Finance manager)
- Sales Access → Manager or Director approval
- Developer Access → Tech Lead or Engineering Manager
- HR Access → HR Director
- Executive Access → CEO or HR Director

**Step 3: Define Time Limits**

- Standard team access: 90 days (review quarterly)
- Contractor access: 6 months
- Executive access: 1 year (less frequent reviews)
- Temporary project access: 30 days

---

## The Scenario Check

**Contoso's entitlement management:**

Alex Lee was hired into the Finance team. She submitted an access request through Contoso's access portal, selecting "Finance Team Access." Her manager River approved within 1 hour. The system automatically added Alex to the Finance-All group and Finance SharePoint site. After 90 days, Alex received a renewal notice. She renewed the access for another 90 days.

When Alex was later transferred to Engineering, her Finance access was set to expire (not renewed), and she requested "Developer Access," which was approved by her new manager.

When a contractor (Sam Taylor) was brought on for a 3-month project, he requested temporary project access. After 90 days, his access automatically expired without anyone having to manually remove him.

This process replaced weeks of email chains and manual steps. Everything is auditable.

---

## Check Your Understanding

- [ ] Explain the difference between manual access assignment and entitlement-managed access
- [ ] Describe what an "access package" is with one example
- [ ] State the benefit of time-limited access vs. standing access
- [ ] Design an access package for your organization
- [ ] Explain why a "catalog" is useful for users requesting access

---

## Common Mistakes

**Creating access packages that are too broad.**
You create an "All Access" package that includes access to all groups, apps, and resources. Anyone can request it. New hires request it. Soon everyone has access to everything. Create focused packages: "Finance Team Access," not "All Access."

**Not setting time limits on access packages.**
Users get access with no expiration. They never renew. Five years later, they still have access to a team they left. Access becomes stale. Always set time limits (90 days is standard). Force renewal as a review gate.

**Approving without question.**
Manager sees access request and approves without reviewing. Alex requests "Executive Access" (should be for executives only). Manager thinks the system auto-approved, doesn't notice it. Approvers should review requests. Risk-based approval rules help: some packages auto-approve, others require human review.

**Not notifying users before expiration.**
Access is about to expire. User never sees the notification (filtered to spam). Access expires without warning. User loses access mid-project. Send multiple notifications: at 60 days, 30 days, 7 days.

**Forgetting to update access packages when roles change.**
Finance team gets a new shared drive. "Finance Team Access" package isn't updated to include the new drive. New hires in Finance don't get the new resource. When roles change, update the packages.

---

## What's Next

Module 8.2 covers **Access Packages** — how to create access packages and add resources to them. Module 8.3 covers **Approval Workflows** — how to configure who approves requests and under what conditions.

---

> 📚 **Entra ID — Zero to Admin** · Unit 8: Identity Governance
>
> [← Unit 7: Privileged Access Management](../../UNIT%207%20-%20PRIVILEGED%20ACCESS%20MANAGEMENT/7-6-pim-breakfix/7-6-pim-breakfix.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 8.2 — Access Packages →](../8-2-access-packages/8-2-access-packages.md)
