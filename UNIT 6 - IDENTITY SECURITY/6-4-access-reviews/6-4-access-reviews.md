# Access Reviews — Who Really Needs Access?
*Every quarter, ask: does this person still need this access? Remove it if they don't. This is compliance and security.*

> 🟡 **Intermediate** · Unit 6, Module 6.4 · ⏱️ 50 minutes

---

## What You Will Learn

- Understand what access reviews are and why they're required for compliance
- Create and manage access reviews for apps and groups
- Review access as a manager or approver
- Remove unnecessary access based on review decisions
- Schedule recurring reviews and track compliance

---

## Why This Matters

*SC-300: Plan and implement identity governance — Manage access reviews and certify access*

Access accumulates. Alex Lee joined Finance, got access to Finance apps. She transferred to Engineering, got Engineering access. She left Engineering, but nobody removed Finance access. Now Alex has access to both Finance and Engineering systems for years. This is called "access creep" — excess permissions that nobody audits. Access reviews fix this: every quarter (or on a schedule), managers review their team's access and confirm it's still needed. If not, it's removed. This is required for compliance (SOC 2, HIPAA, etc.) and strengthens security.

---

## 📋 What Access Reviews Cover

Access reviews can audit:

| Type | What's reviewed | Example |
|---|---|---|
| **App access** | Who can access which apps | Does Alex still need Salesforce? Does Morgan still need GitHub? |
| **Group membership** | Who's in which groups | Is Jordan still in "Managers" group? Is Sam (contractor) still in "Employees"? |
| **Resource access** | Who can access files/data | Does River still need access to HR shared folders? |
| **Privileged roles** | Who has admin roles | Does Alex (Finance) need to be a Global Administrator? |

---

## 🔄 How Access Reviews Work

**Phase 1: Initiate Review**
- Admin creates a review: "Review Finance team's access to Salesforce"
- Assigns **reviewers** (usually managers): River Patel will review Finance access
- Sets **scope**: "All users in Finance group"
- Sets **deadline**: "Decisions due by March 31"

**Phase 2: Reviewers Decide**
- Reviewer (River) receives notification: "Please review your team's access"
- For each user, reviewer approves: "Yes, Morgan still needs Salesforce" or denies: "No, Alex no longer needs it"
- Reviewer completes the review before deadline

**Phase 3: Auto-apply or Manual Action**
- **Auto-apply:** Entra ID automatically removes denied access
- **Manual:** Admin reviews the decisions and manually removes access

**Phase 4: Document and Compliance**
- Report generated: "Access review completed, X users approved, Y removed"
- Saved for compliance audits: "We reviewed access quarterly, here's proof"

---

## Lab 6.4.1 — Create an Access Review

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 5.6 complete (users assigned to apps)

> 🔑 **Licensing:** Access Reviews require **Entra ID P2**. Your dev tenant includes this.

1. Navigate to **Identity Governance** → **Access reviews** (or **Governance** → **Access reviews**).

2. Click **+ New access review**.

3. **Name:** `Finance Team Salesforce Access Review`

4. **Description:** `Quarterly review of Finance team access to Salesforce`

5. **Select what to review:**
   - Choose **Apps and service principals** (reviewing app access)
   - Select **Salesforce** (or any app you've configured)

6. **Scope:**
   - Select **Groups** → Choose **Finance-All** group (or Finance users)
   - This means: Review access for all users in the Finance-All group

7. **Reviewers:**
   - Select **Group owner** (if available)
   - Or manually select **River Patel** (HR Manager, suitable reviewer)

8. **Settings:**
   - **Duration:** Set to **1 month** (deadline for review completion)
   - **Auto-apply results:** Toggle **On** (auto-remove denied access)
   - **Send reminders:** Toggle **On** (remind reviewers before deadline)

9. Click **Create**.

The review is now active. River Patel will receive a notification to review Finance users' Salesforce access.

---

## Lab 6.4.2 — Complete an Access Review (As Reviewer)

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 6.4.1 complete; at least one user in the review scope

In this lab, you'll simulate being the reviewer (River Patel) and deciding on access.

1. Navigate to **Identity Governance** → **Access reviews**.

2. Find the review you created: `Finance Team Salesforce Access Review`.

3. Click on it. You'll see **Pending decisions** section with a list of users.

4. For each user (Alex Lee, Morgan Chen, etc.):
   - Read their name and current access
   - Decide: Does this user still need Salesforce access?

5. For **Alex Lee** (Finance Analyst):
   - She's still in Finance
   - Approve: **Yes, still needs access**

6. For **Morgan Chen** (Software Engineer, moved to Engineering):
   - She's no longer in Finance
   - Deny: **No, remove access**

7. For each user, click **Approve** or **Deny**, and optionally add a comment: "Why is this decision made?"

8. Once you've reviewed all users, click **Submit** or **Complete** (button varies).

The review is submitted. If auto-apply is enabled, Entra ID will now remove denied access.

---

## Lab 6.4.3 — Review and Apply Access Review Results

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 6.4.2 complete; review has been submitted

1. Navigate to **Access reviews** and open the completed review.

2. You'll see **Review results**:
   - Total users reviewed
   - Approved (still need access)
   - Denied (removed access)
   - Not reviewed (deadline passed, no decision made)

3. Click on the review to see detailed results:
   - **User** | **Decision** | **Reason**
   - Alex Lee | Approved | Finance Analyst, still in Finance
   - Morgan Chen | Denied | No longer in Finance group
   - Etc.

4. If auto-apply is **Off**, you must manually remove denied access:
   - For each denied user, navigate to their app assignment
   - Click **Remove assignment**

5. If auto-apply is **On**, Entra ID has already removed access. Verify by:
   - Navigating to **Salesforce** → **Users and groups**
   - Confirming denied users are no longer assigned

6. Download or export the review report (if available):
   - This becomes compliance documentation
   - "We reviewed access on [date], results: [approved/denied counts]"

---

## Lab 6.4.4 — Schedule Recurring Access Reviews

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 6.4.3 complete

Set up a quarterly access review that repeats automatically.

1. Navigate to **Access reviews** → **+ New access review**.

2. **Name:** `Quarterly Finance Access Review`

3. **Select what to review:** Apps → Salesforce

4. **Scope:** Group → Finance-All

5. **Reviewers:** River Patel

6. **Recurrence:**
   - Toggle **Enable recurrence** to **On**
   - Select **Quarterly** (every 3 months)
   - Start date: Today

7. **Duration:** 1 month per review cycle

8. **Auto-apply:** On

9. Click **Create**.

Now every quarter, a new access review is automatically created. River Patel will get a reminder every quarter to review Finance access. If she denies access, it's automatically removed.

---

## The Scenario Check

Contoso has implemented quarterly access reviews. Every three months, managers review their teams' access:

**Q1 Review (January):** River reviews Finance access. Alex (still in Finance) is approved. Morgan (transferred to Engineering) is denied — her Salesforce access is removed.

**Q2 Review (April):** River reviews again. The team composition hasn't changed. Everyone still needs their current access. All approved.

**Q3 Review (July):** Sam Taylor (contractor) is removed from Contoso. The review shows Sam in the Finance access list. River denies Sam's access. It's automatically removed that day.

**Compliance Audit (December):** Auditors ask: "Prove you review access regularly." Contoso shows 4 quarterly reviews from the past year, each with documented decisions and removals. Compliance confirmed.

---

## Check Your Understanding

- [ ] Navigate to **Access reviews** and create a new review (don't submit, just set it up)
- [ ] For an access review, explain the difference between "Approve" and "Deny"
- [ ] State what auto-apply does when you deny access
- [ ] Describe why quarterly access reviews are a compliance requirement
- [ ] Explain what "access creep" is and how access reviews prevent it

---

## Common Mistakes

**Not setting a deadline for access reviews.**
You create a review with no deadline. Reviewers never complete it. Access is never removed. Set a deadline 2-4 weeks out so reviewers have time but aren't dragging their feet.

**Making access review scope too large.**
You create a review for "All users in the entire tenant." The reviewer is overwhelmed — 500 users to review, gives up. Create focused reviews: "Finance team's Salesforce access" not "everyone's everything."

**Not following up on denied access.**
Auto-apply is Off. You deny access for 10 users. You never manually remove it. Those users still have access. If auto-apply is off, assign someone to remove the denied access.

**Reviewing without understanding the role.**
Reviewer hasn't met the user. "Does Sales-Team-Lead need Salesforce?" Reviewer guesses. Ask reviewers to be the user's manager or someone who knows their role, not a random admin.

**Forgetting to schedule recurring reviews.**
You create one-time access review. You remember to do it this quarter. Forget next quarter. Set recurring reviews so they happen automatically.

---

## What's Next

Module 6.5 covers **Incident Response — Compromised Accounts** — the step-by-step playbook for when an account is actively compromised and in use by an attacker. You'll learn immediate containment, investigation, and recovery. Module 6.6 is a breakfix lab where you respond to a simulated incident.

---

> 📚 **Entra ID — Zero to Admin** · Unit 6: Identity Security
>
> [← Module 6.3 — Risky Users and Remediation](../6-3-risky-users-remediation/6-3-risky-users-remediation.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 6.5 — Incident Response →](../6-5-incident-response/6-5-incident-response.md)
