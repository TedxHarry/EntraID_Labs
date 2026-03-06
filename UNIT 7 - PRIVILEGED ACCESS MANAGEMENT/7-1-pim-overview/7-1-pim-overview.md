# PIM Overview — Control Who Becomes Admin and When
*Not everyone should be admin all the time. Privileged Identity Management (PIM) is the guard that controls access to dangerous permissions.*

> 🟡 **Intermediate** · Unit 7, Module 7.1 · ⏱️ 50 minutes

---

## What You Will Learn

- Explain what Privileged Identity Management is and why it matters
- Understand the difference between permanent admin roles and eligible roles
- Describe PIM's four core capabilities (eligibility, approval, activation, auditing)
- Identify scenarios where PIM prevents accidental or malicious privilege escalation
- Understand just-in-time access and why it reduces risk

---

## Why This Matters

*SC-300: Plan and automate identity governance — Implement Privileged Identity Management (PIM)*

Units 1–6 taught you to build identity infrastructure and defend it from attacks. Unit 7 teaches you to control *privileged access itself*. A compromised global admin account can change every user's password, create hidden admins, and steal data. PIM reduces this risk by restricting who can be admin, requiring approval to become admin, and auditing everything admins do. In regulated industries (healthcare, finance, government), PIM is mandatory. In enterprises, it's a best practice.

---

## 🔐 The Problem: Standing Privileges

**Current state (bad):** Alex is a Global Administrator. She's admin 24/7. She can do anything, anytime. If her account is compromised, the attacker has full tenant access immediately.

**Scenario of what an attacker could do with Global Admin:**
- Read all users' emails
- Reset all passwords
- Delete all groups and teams
- Create hidden admin accounts
- Grant themselves Exchange Online admin
- Export all data
- Disable all security settings

**The problem:** Even if the attacker doesn't do all of this immediately, an admin account in an attacker's hands is catastrophic. And if compromised, you might not know it immediately.

---

## ✅ The Solution: PIM (Privileged Identity Management)

**PIM principles:**
1. **Minimal admin time** — Users are admins only when needed (minutes to hours, not weeks)
2. **Approval required** — Activating admin access requires approval from another admin
3. **Audited** — Every admin action is logged and can be investigated
4. **Constrained** — Time limits (2 hours max), MFA required, conditional access applies

**Result:** If an attacker compromises an admin account, they can't immediately access admin tools. They must request activation, wait for approval, complete MFA, and all their actions are logged.

---

## 📋 Four Core Concepts

### 1. **Eligible vs. Permanent Admin**

**Permanent Admin (Bad):**
- User is admin all the time
- No approval needed
- No time limit
- If compromised: Immediate full access

**Eligible Admin (Good):**
- User *can become* admin, but isn't admin by default
- Must request activation
- Activation requires approval (or immediate if no approval required)
- Access is time-limited (2 hours, 8 hours, custom)
- If compromised: Attacker must request activation, wait for approval, answer MFA

**Example:**
- **Permanent:** Jordan (IT Admin) has "Global Administrator" role permanently
- **Eligible:** Jordan is *eligible* for "Global Administrator" but isn't admin until they activate
  - Jordan: "I need Global Admin access for 2 hours to reset a user's password"
  - PIM: "Request submitted to River Patel for approval"
  - River: "Approved ✓"
  - Jordan: "Please activate my Global Admin access"
  - PIM: "Activated for 2 hours (until 2:15 PM). Your actions will be audited."
  - Jordan gets admin access, makes the password change, and access expires automatically

---

### 2. **Activation**

**What it means:** Turning eligible access into active access.

**Process:**
- User logs in normally
- User navigates to PIM
- User requests activation: "I want to activate Global Admin for 2 hours"
- PIM asks for **reason** (optional but encouraged): "Resetting user passwords for offboarding"
- If approval is required: Request goes to approver, user waits for approval
- If no approval required: Activation is instant (if under approval threshold)
- User completes MFA (extra verification)
- Role is activated for the requested duration

**Example timeline:**
- 12:00 PM: Jordan requests activation
- 12:05 PM: River approves
- 12:06 PM: Jordan completes MFA
- 12:07 PM: Jordan's Global Admin access is active (until 2:07 PM)
- 2:07 PM: Access expires automatically

---

### 3. **Approval**

**Who approves?** Typically, a senior admin (manager, security team, another admin in the role).

**When is approval required?**
- High-privilege roles (Global Admin, Security Admin, Cloud Application Admin)
- Activation requests from high-risk users (users not seen activating often)
- Manual approval can be configured in PIM settings

**What does an approver see?**
- User requesting access: Jordan Kim
- Role requested: Global Administrator
- Duration: 2 hours
- Reason: "Password resets for departing employee"
- Approver decision: Approve or Deny

**If denied:** User is notified. They can request again, or escalate to another approver.

---

### 4. **Auditing**

**What gets logged:**
- Every activation request (time, user, reason)
- Every approval decision (who approved, when, any comments)
- Every activation (when, duration, MFA result)
- Every admin action taken while in PIM role (reading data, making changes)
- Every deactivation or expiration

**Use cases:**
- "Who activated Global Admin on 3/5 at 2 PM?" → Audit log shows Jordan
- "What did Jordan do after activating?" → Session logs show password reset for user #1234
- "How often is Global Admin activated?" → Audit report shows 5 activations/month (too often?)
- "Compliance audit: Prove admins are approved" → Run PIM audit report for auditors

**Retention:** Entra ID keeps audit logs for 30 days (free tier) or longer with premium logging.

---

## 🎯 PIM Use Cases

### Use Case 1: Prevent Accidental Damage

**Scenario:** Alex is a Global Admin. She's tired at 5 PM on a Friday. She fat-fingers a command and deletes a critical group. Users lose access to their team channel. Recovery takes hours.

**With PIM:** Alex is eligible for Global Admin, not permanent. She rarely needs it. Before she can become admin at 5 PM Friday, PIM:
- Requires approval (River is out of office, can't approve in 15 minutes)
- Or: Activation is approved, but PIM logs every action (deletion is visible in audit, can be restored)

**Result:** Either access is denied, or the risky action is visible immediately.

---

### Use Case 2: Reduce Blast Radius if Account Compromised

**Scenario:** An attacker compromises Alex's account (Global Admin). They immediately grant themselves Exchange Online admin and start exfiltrating emails.

**Without PIM:** Attacker is admin all the time. By the time you notice, they've stolen months of emails.

**With PIM:** Attacker has Alex's password but is not admin. To become admin, they must:
- Request activation (you get an alert: "Unusual activation request from Tokyo at 2 AM")
- Wait for approval (no one is awake; approval is denied)
- Or: PIM grants activation, but all actions are audited (you see attacker's actions immediately, can revoke access)

**Result:** Attack window is measured in minutes (until you deny approval), not hours/days.

---

### Use Case 3: Compliance Requirement

**Scenario:** A compliance auditor asks: "Prove no one has standing admin access to sensitive systems."

**Without PIM:** You have 15 Global Admins with permanent access. Auditor is concerned.

**With PIM:** You have 15 users *eligible* for Global Admin, but none are permanent admins. Every activation is approved and logged. Auditor is satisfied: "Access is controlled and audited."

**Result:** You pass compliance audits.

---

## 📊 PIM Architecture

```
User wants admin access
       ↓
Is user eligible for the role?
  ├─ No → Denied
  └─ Yes → Proceed
       ↓
User requests activation
       ↓
Is approval required for this role?
  ├─ No → Proceed to MFA
  └─ Yes → Send to approver
       ↓
Approver reviews request
       ├─ Denies → User is notified, request ends
       └─ Approves → Proceed
       ↓
User completes MFA
       ↓
Role is activated for configured duration
       ↓
PIM monitors user's admin actions
       ↓
Duration expires OR user deactivates manually
       ↓
Role deactivated, audit logged
```

---

## 🔑 Key Entra PIM Features

| Feature | What It Does |
|---|---|
| **Eligibility** | User is eligible for a role (can become admin) but not admin by default |
| **Activation** | User activates eligible access, with optional approval and MFA |
| **Time limits** | Admin access expires automatically (1 hour, 2 hours, custom, 8 hours max) |
| **Approval** | High-privilege roles require another admin's approval before activation |
| **MFA enforcement** | User must complete MFA again before activating (extra verification) |
| **Conditional Access** | CA policies apply to PIM activations (location, device, IP, time of day) |
| **Auditing** | Every request, approval, activation, and action is logged |
| **Alerts** | Get notified of suspicious PIM activity (unusual times, unusual users, unusual duration) |

---

## Lab 7.1.1 — Understand PIM Licensing

**Time:** 5 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 1.1 complete

> 🔑 **Licensing:** PIM requires **Entra ID P2**. Your developer tenant includes this. In production, verify you have P2 before enabling.

1. Navigate to **Identity Governance** → **Privileged Identity Management** (or search "PIM").

2. You should see the PIM dashboard. If you see "License required," your tenant doesn't have P2.

3. For this learning series, assume P2 is available (dev tenant includes it).

---

## Lab 7.1.2 — Explore PIM Dashboard

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.1.1 complete

1. Navigate to **Privileged Identity Management** → **Overview** (or **Privileged access management** → **Identity Governance**).

2. The PIM dashboard shows:
   - **Quick stats:** Number of eligible users, permanent admins, pending approvals
   - **Roles:** All roles you can manage with PIM
   - **Alerts:** PIM security alerts
   - **Recent activity:** Recent activations, approvals, actions

3. On the left menu, you'll see sections:
   - **Azure AD roles** — Manage Entra ID admin roles (Global Admin, User Admin, etc.)
   - **Azure resources** — Manage access to Azure subscriptions/resources (not part of SC-300 focus)
   - **Approval** — Review and approve/deny activation requests
   - **Activity** — Audit logs of all PIM actions
   - **Alerts** — PIM security alerts

4. Spend 5 minutes exploring each section. No action needed yet — just familiarize yourself with the layout.

---

## Lab 7.1.3 — Review Current Admin Roles

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 7.1.2 complete

Understand what admin roles currently exist in your tenant and who has them.

1. Navigate to **Privileged Identity Management** → **Azure AD roles** → **Roles**.

2. You'll see a list of roles:
   - Global Administrator
   - Application Administrator
   - Cloud Application Administrator
   - User Administrator
   - Etc.

3. For each role, note:
   - **Active assignments:** How many users are permanent admins in this role?
   - **Eligible assignments:** How many users are eligible but not permanent?
   - **Example:** Global Administrator might show 1 active (permanent), 2 eligible

4. Click on **Global Administrator** role.

5. You'll see:
   - **Active assignments:** Users who are admins right now
   - **Eligible assignments:** Users who can become admins but aren't

6. For each user in "Active assignments":
   - Are they truly supposed to be permanent admins?
   - Could they be eligible instead (activate when needed)?

7. **Action:** Identify one permanent admin who could be converted to eligible in Module 7.2.

---

## Lab 7.1.4 — Understand Just-in-Time (JIT) Access

**Time:** 10 minutes
**Tools:** Conceptual
**Prerequisite:** Lab 7.1.3 complete

Just-in-time access is the core concept of PIM: access is granted only when needed, for as long as needed.

### Example: Just-in-Time vs. Standing Access

**Without JIT (Standing Admin):**
- Jordan is Global Admin 24/7/365
- Risk: If Jordan's account is compromised on Tuesday, attacker has admin access through Friday (until detected)

**With JIT (PIM Eligible):**
- Jordan is eligible but not admin by default
- Monday 9 AM: Jordan activates Global Admin for 2 hours (to add a new user)
- Monday 11 AM: Access expires, Jordan is no longer admin
- Monday 2 PM: Jordan requests another 1-hour activation (to reset a password)
- Monday 2:05 PM: Approval is granted, access is activated for 1 hour
- Monday 3 PM: Access expires again
- Jordan's admin time is measured in hours per week, not 24/7

**Attacker scenario with JIT:**
- If Jordan's account is compromised Tuesday 2 PM, and Jordan isn't currently admin:
  - Attacker would need to request activation (you get alert)
  - If approval is required, attacker waits for approval (you have time to deny)
  - If attacker tries to force activation: Fails without approval, or MFA blocks them

**Result:** Attack window is shrunk from 72 hours (standing admin) to minutes (JIT activation time).

---

## The Scenario Check

At Contoso, Jordan Kim is the IT administrator. Previously, she was a permanent Global Administrator — admin all the time. Under PIM:

- Jordan is now *eligible* for Global Administrator role (can become admin, but isn't by default)
- When Jordan needs admin access (to reset a user's password, add a new user, etc.), she requests activation through PIM
- River Patel (HR Manager, also an admin) approves the request
- Jordan activates access for the minimum duration needed (1–2 hours)
- All of Jordan's admin actions are logged
- Access expires automatically

This change reduces risk: If Jordan's account is compromised, the attacker can't immediately use it as admin. They must request activation, wait for approval, and all actions are visible to auditors.

---

## Check Your Understanding

- [ ] Explain the difference between "eligible" and "permanent" admin roles
- [ ] Describe what "activation" means in PIM
- [ ] State why approval is important for high-privilege roles
- [ ] Explain how PIM reduces risk if an admin account is compromised
- [ ] Describe one scenario where JIT access prevents damage

---

## Common Mistakes

**Converting all admins to PIM overnight.**
You enable PIM and make everyone eligible instead of permanent. Suddenly, no one can do urgent admin tasks because activations require approval, and approvers are slow. Admins get frustrated. Transition to PIM gradually: start with high-privilege roles (Global Admin), then expand to other roles.

**Setting approval requirement for every role.**
You require approval for all role activations, including low-risk roles like "User Administrator" when the user count is being managed by multiple admins. Activation requests get backed up. Approvers are overwhelmed. Require approval only for high-risk roles (Global Admin, Security Admin, Exchange Online Admin). Allow low-risk roles to activate without approval.

**Not setting time limits for activations.**
A user activates "Global Administrator" for 8 hours (the maximum). After 6 hours, they forget to deactivate. They're admin longer than necessary. Set reasonable time limits: 1–2 hours for password resets, 4 hours for more complex tasks. Users can request extensions if needed.

**Ignoring PIM alerts.**
PIM generates alerts (suspicious activation requests, unusual times, unusual roles). You don't monitor them. A malicious insider requests Global Admin access from a foreign IP at 3 AM — alert is generated but ignored. Designate someone to monitor PIM alerts daily.

**Not training users on PIM activation process.**
You enable PIM, but admins don't know how to request activation. They email IT: "I need admin access." Support team has to walk them through PIM. Provide training and documentation on the activation process before enabling.

---

## What's Next

Module 7.2 covers **Configuring PIM** — how to set up PIM in your tenant. You'll configure which roles are eligible, which require approval, time limits, MFA requirements, and notification settings. Module 7.3 walks through activation from a user's perspective.

---

> 📚 **Entra ID — Zero to Admin** · Unit 7: Privileged Access Management
>
> [← Unit 6: Identity Security](../../UNIT%206%20-%20IDENTITY%20SECURITY/6-6-incident-response-breakfix/6-6-incident-response-breakfix.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 7.2 — Configuring PIM →](../7-2-pim-configuration/7-2-pim-configuration.md)
