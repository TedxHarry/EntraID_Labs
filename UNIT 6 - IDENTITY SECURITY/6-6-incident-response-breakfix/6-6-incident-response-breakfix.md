# Breakfix: Incident Response Lab
*An account is actively compromised. You have the logs. You have the tools. Respond to a real-looking incident from detection to recovery.*

> 🔴 **Advanced** · Unit 6, Module 6.6

---

## What You Will Learn

- Apply the full incident response playbook to a realistic scenario
- Prioritize actions during active compromise (what to do first, second, third)
- Investigate using sign-in logs, audit logs, and risk detections
- Make containment vs. investigation trade-off decisions
- Document findings and create preventative controls

---

## Why This Matters

*SC-300: Implement authentication and access management — Execute incident response procedures for compromised accounts*

Modules 6.1–6.5 taught you incident response techniques individually. This lab combines them into a realistic, time-pressured scenario. In real incidents, you'll face incomplete information, tight timelines, and decisions that balance security with usability. This lab teaches you to prioritize like an experienced incident responder.

---

## The Scenario: Live Incident

**Alert:** Your SOC (Security Operations Center) sends this notification:

> **SECURITY ALERT — Possible Account Compromise**
>
> Account: Morgan Chen (morgan.chen@contoso.com)
> Risk Level: HIGH
> Reason: Impossible travel detected (New York sign-in, then Tokyo sign-in, 10 minutes apart)
> Last activity: 2:10 PM Tokyo time, Outlook access
> Recommended action: Immediate investigation and containment

Your job: Respond to this incident. You have 90 minutes to:
1. Confirm the compromise (is it real or false alarm?)
2. Contain it (stop the attacker)
3. Investigate (understand scope and damage)
4. Recover (restore account)
5. Strengthen (prevent recurrence)

---

## Your Decision: Containment First or Investigation First?

**Decision point:** Should you immediately revoke sessions (containment) or first investigate sign-in logs (understand before acting)?

**The dilemma:**
- **Immediate containment (revoke sessions):** Stops the attacker now, but might interrupt legitimate work if it's a false alarm
- **Investigate first:** Understand the situation before taking action, but the attacker has more time to cause damage

**Real-world answer:** Revoke sessions *immediately* for high-risk indicators like impossible travel. You can restore access in 30 seconds if it's a false alarm. You cannot undo data theft if you delay. **Containment first, questions later.**

Your first action: **Revoke Morgan's sessions.**

---

## Lab 6.6.1 — Immediate Containment (Minute 1)

**Tools:** Entra admin center
**Scenario context:** You've just received the alert. Assume impossible travel detection is high confidence. Act now.

### Step 1: Revoke All Sessions

1. Navigate to **Users** → **All users**.

2. Search for **Morgan Chen**.

3. Click on Morgan's profile.

4. Click **Sign-out sessions** (revoke all active sessions).

5. Confirm the action.

**Result:** Any active attacker sessions are terminated. Morgan is signed out everywhere. If this is a false alarm (legitimate travel), Morgan can sign in again immediately with their password.

### Step 2: Reset Password (Attacker's Password is Invalid)

1. On Morgan's profile, click **Reset password**.

2. Select **Generate temporary password** (Entra ID creates a random secure password).

3. Copy the temporary password. **Do not email it** — call Morgan directly or use secure SMS.

4. Check: **User must change password on next sign-in** ✓

5. Click **Reset**.

**Result:** The stolen password (if the attacker has it) is now invalid. The temporary password is known only to you and Morgan. The attacker is locked out.

### Step 3: Preliminary Assessment (Is it Real?)

Immediate question: Is this a real compromise or a false alarm (legitimate travel, unusual device)?

- If Morgan was on a plane from New York to Tokyo, the "impossible travel" is actually a false alarm
- If Morgan was in New York all day and never traveled, the sign-ins are real compromise

**Action:** Quickly contact Morgan or check her calendar:
- Was she traveling? ← If yes, false alarm, re-enable access, tighten monitoring
- Was she in NYC? ← If yes, confirm compromise, proceed with investigation

**For this scenario, assume Morgan was in NYC all day. The compromise is real.**

---

## Lab 6.6.2 — Investigation Phase 1: Sign-In Logs (Minutes 5–20)

**Tools:** Entra admin center
**Scenario context:** Morgan confirms she was in NYC all day. She didn't sign in from Tokyo. The compromise is real.

### Step 1: Review Sign-In Timeline

1. Navigate to **Users** → **All users** → **Morgan Chen** → **Sign-in logs** (or **Monitoring** → **Sign-in logs**, filter by Morgan).

2. Set date range: **Last 24 hours**.

3. Review the timeline. You should see entries like:

   | Time | Location | Device | Result | Notes |
   |---|---|---|---|---|
   | 9:00 AM | NYC, Office IP | Windows laptop | Success | ✓ Normal |
   | 12:30 PM | NYC, Office IP | Windows laptop | Success | ✓ Normal |
   | 1:55 PM | Tokyo, Unknown IP | Chrome browser | Success | ⚠️ Suspicious |
   | 2:00 PM | Tokyo, Unknown IP | Chrome browser | Success | ⚠️ Suspicious |
   | 2:05 PM | Tokyo, Unknown IP | Chrome browser, Outlook access | Success | ⚠️ Suspicious |
   | 2:10 PM | Tokyo, Unknown IP | Chrome browser | Success | ⚠️ Suspicious |

4. **Key questions to answer:**
   - **When did the compromise start?** (first Tokyo sign-in at 1:55 PM)
   - **How did they bypass MFA?** (if Morgan has MFA, how did attacker sign in? Answer: Maybe they have her phone, or MFA can be disabled if they have sufficient permissions)
   - **How long were they in?** (1:55 PM to 2:10 PM = ~15 minutes before we revoked sessions)
   - **What did they access?** (Outlook, Teams, SharePoint — all Microsoft services)

5. **Document:** Create a simple timeline:
   - Compromise window: 1:55 PM – 2:15 PM (when you revoked sessions) = 20 minutes of access

### Step 2: Determine Attack Source

1. Look at the **IP address** of the Tokyo sign-ins:
   - Example: `203.0.113.45` (hypothetical)
   - This is a residential IP in Tokyo, Japan

2. **Question:** Is this a real attacker in Tokyo, or a compromised device nearby that forwarded traffic?
   - If it's a residential IP: Likely a real person in Tokyo with stolen credentials
   - If it's a VPN service: Attacker is hiding location, could be anyone
   - If it's a data center: Attacker is likely using automated tools (credential stuffing, testing stolen passwords)

3. For this scenario, assume it's a **residential Tokyo IP** — likely a user/attacker account sold on dark web.

### Step 3: Check for MFA Bypass

1. Review the Tokyo sign-in details:
   - Was MFA required and completed?
   - Was MFA bypassed?
   - Was the device marked "trusted" (so MFA wasn't required)?

2. If MFA was **not required**, the attacker didn't need to intercept MFA codes. This might mean:
   - Morgan doesn't have MFA enabled (bad)
   - The attacker had Morgan's password + her phone (they stole both)
   - MFA was bypassed through a vulnerability (rare)

3. **Action:** Check Morgan's MFA settings — does she have it enabled? If not, this is a contributing factor to the compromise.

---

## Lab 6.6.3 — Investigation Phase 2: Audit Logs (Minutes 20–35)

**Tools:** Entra admin center
**Scenario context:** You understand the sign-in timeline. Now investigate what the attacker *did*.

### Step 1: Review Changes to Morgan's Account

1. Navigate to **Audit logs** (or **Monitoring** → **Audit logs**).

2. Filter:
   - **User:** Morgan Chen
   - **Activity:** All activities
   - **Date:** Last 24 hours

3. Look for suspicious changes. You might see:

   | Time | Activity | Details | Severity |
   |---|---|---|---|
   | 1:58 PM | User sign-in | Tokyo IP, Outlook | High |
   | 2:00 PM | User updated | Email forwarding rule added | High |
   | 2:03 PM | User updated | Mailbox delegate added (attacker@external.com) | Critical |
   | 2:05 PM | Group membership changed | Morgan added to "Finance Admins" group | Critical |

4. **Critical findings:**
   - **Email forwarding rule:** Attacker set rule "Forward all emails to attacker@external.com"
     - Every email Morgan receives is now copied to the attacker
     - Attacker has persistent access to communications
   - **Mailbox delegate:** Attacker added their own account as delegate
     - Attacker can read all of Morgan's past and future emails
     - Attacker can send emails as Morgan
   - **Group membership:** Morgan was added to "Finance Admins"
     - This is unusual (Morgan is Engineering, not Finance)
     - Attacker was trying to escalate privileges to access Finance data

### Step 2: Assess Damage

Based on audit logs, what was the attacker trying to do?

- **Persistent access:** Email forwarding + delegate = attacker can read emails even after password reset
- **Privilege escalation:** Added to Finance Admins group = attacker wanted access to Finance systems
- **Data theft:** Likely attempted to access Finance data, customer information, deals in progress

**Severity assessment:** This is a HIGH severity incident. The attacker had 20 minutes and set up multiple persistence mechanisms.

### Step 3: Check Application Access

1. In **Audit logs**, filter for **App created** or **Service principal created**:
   - Did the attacker create a new app?
   - Did they grant an app new permissions?

2. If you find suspicious app activity:
   - Record the app name and ID
   - Note what permissions it was granted
   - Plan to revoke it in the Recovery phase

---

## Lab 6.6.4 — Investigation Phase 3: Scope (Minutes 35–50)

**Tools:** Entra admin center, Azure
**Scenario context:** You've documented Morgan's compromise. Now investigate: Is Morgan alone, or are other accounts compromised?

### Step 1: Check for Other Impossible Travel Detections

1. Navigate to **Protection** → **Identity Protection** → **Risky sign-ins**.

2. Filter by **Risk level: High** and **Date: Last 24 hours**.

3. Look for other users with impossible travel or suspicious patterns around the same time as Morgan's compromise:
   - 1:50 PM – 2:20 PM timeframe

4. If you see other users with suspicious activity at the same time, this might indicate:
   - A compromised credential database (attacker has multiple passwords)
   - A credential stuffing attack (attacker trying many passwords)
   - Lateral movement (attacker using Morgan's account to access others)

5. **For this scenario:** Assume only Morgan shows impossible travel. Other users are not directly compromised (but check in real incidents).

### Step 2: Check for Hidden Admin Creation

An attacker with access might create a hidden admin account to maintain persistence. Check:

1. Navigate to **Users** → **All users**.

2. Look for any users created in the last 24 hours with unusual names (e.g., "admin1", "service_account", "svc_morgan").

3. If you find suspicious users:
   - Note their details (ID, email)
   - Plan to delete them in Recovery phase

4. **For this scenario:** Assume no hidden accounts were created.

### Step 3: Check for Application Consent Grants

1. Navigate to **Applications** → **Permissions**.

2. Look for any apps with recent permission grants (last 24 hours).

3. If the attacker created or granted permissions to a suspicious app:
   - Document the app name
   - Note the permissions granted (especially high-risk ones like `User.ReadWrite.All`)
   - Plan to revoke in Recovery phase

---

## Lab 6.6.5 — Recovery: Undo Attacker Changes (Minutes 50–70)

**Tools:** Entra admin center, Exchange admin center
**Scenario context:** You've contained and investigated. Now repair the damage.

### Step 1: Remove Mailbox Delegate

The attacker added themselves as a delegate. Remove them:

1. Navigate to **Exchange Admin Center** (or Outlook → **Settings** → **Mail** → **Forwarding**).

2. Find **Morgan Chen** → **Mailbox settings** → **Delegates** (or similar).

3. Look for the attacker's account (e.g., attacker@external.com):
   - Click **Remove** or **Delete**

4. **Result:** Attacker can no longer read/send emails as Morgan.

### Step 2: Remove Email Forwarding Rule

1. In **Exchange Admin Center**, find **Morgan Chen** → **Mail rules** (or **Outlook** → **Settings** → **Forwarding**).

2. Look for a rule like "Forward all mail to attacker@external.com":
   - Click **Delete** or **Disable**

3. **Result:** Morgan's emails are no longer copied to the attacker.

### Step 3: Remove Morgan from Unauthorized Groups

1. Navigate to **Groups** → **All groups** → **Finance Admins** (or the suspicious group).

2. Find **Morgan Chen** in the group members.

3. Click **Remove member**.

4. **Result:** Morgan is no longer in Finance Admins. She can no longer access Finance systems.

### Step 4: Restore Deleted Items (If Any)

If the attacker deleted emails:

1. In **Exchange Admin Center**, find **Morgan** → **Recover deleted items** (or **Mailbox recovery**).

2. Restore any deleted items if available (Exchange retention allows recovery for 30 days).

3. **Result:** Deleted emails are restored.

---

## Lab 6.6.6 — Strengthen: Prevent Recurrence (Minutes 70–85)

**Tools:** Entra admin center
**Scenario context:** Morgan's account is now secured. Prevent this from happening to Morgan or anyone else.

### Step 1: Enable MFA on Morgan's Account

1. Navigate to **Users** → **All users** → **Morgan Chen** → **Authentication methods**.

2. **Enable MFA:**
   - If available, toggle **Require multifactor authentication** to **Yes**
   - Or navigate to **Multi-factor authentication** and set Morgan to **Enforced**

3. **Result:** Morgan must now use a second factor (phone, app, hardware key) to sign in. Even if the attacker gets a password, they need her phone.

### Step 2: Deploy Passwordless Authentication

If your tenant supports it, enable passwordless for Morgan:

1. In **Authentication methods**, enable **Windows Hello for Business** or **Authenticator app (passwordless)**.

2. Encourage Morgan to enroll in one of these methods.

3. **Result:** Morgan can eventually sign in without a password, eliminating password-based attacks.

### Step 3: Create a Conditional Access Policy to Detect Impossible Travel

Prevent this attack from succeeding in the future:

1. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

2. **Name:** `Block Impossible Travel`

3. **Assignments:**
   - **Users:** All users (or Finance team, or group containing Morgan)
   - **Target resources:** All cloud apps

4. **Conditions:**
   - **Sign-in risk:** High
   - (If available) **Impossible travel:** Configured

5. **Access control:**
   - **Grant:** Require multifactor authentication (OR Block access in enforcement, Report-only for testing)

6. **State:** **Report-only** (initially, to test without breaking legitimate travel)

7. Click **Create**.

**Result:** Future impossible travel attempts will require MFA or be blocked. An attacker cannot simply use a stolen password from abroad.

### Step 4: Review Company-Wide Security Posture

Take this opportunity to strengthen the entire organization:

1. **Question 1:** How many users in your tenant have MFA enabled?
   - Command (if using PowerShell): `Get-MsolUser -All | Where-Object { $_.StrongAuthenticationMethods.Count -eq 0 } | Measure-Object`
   - Or check reports in Entra admin center

2. **Question 2:** Do you have Identity Protection enabled company-wide?
   - If not, enable it (Module 6.1)

3. **Question 3:** Are there other users with weak authentication who might be targeted next?
   - Create a task to enable MFA for all users in high-risk roles (Finance, HR, IT)

---

## Lab 6.6.7 — Documentation and Communication (Minutes 85–90)

**Tools:** Word document or notepad
**Scenario context:** Incident is contained and recovered. Document it.

### Incident Report Template

Create a brief incident report for your records:

```
INCIDENT REPORT
Date: [Date of incident]
Reporter: [Your name]
Affected User: Morgan Chen (morgan.chen@contoso.com)

TIMELINE:
- 1:55 PM: First Tokyo sign-in detected (impossible travel)
- 2:10 PM: Alert received
- 2:15 PM: Sessions revoked, password reset (CONTAINMENT)
- 2:20 PM: Sign-in logs reviewed, compromise confirmed
- 2:35 PM: Audit logs reviewed, attacker changes identified
- 2:45 PM: Mailbox delegate and forwarding rule removed
- 2:55 PM: MFA enabled, CA policy created
- 3:00 PM: User notified

WHAT THE ATTACKER DID:
- Signed in for ~20 minutes from Tokyo IP
- Added attacker@external.com as mailbox delegate
- Created email forwarding rule to external address
- Added Morgan to Finance Admins group

DAMAGE ASSESSMENT:
- Exposure: Morgan's emails and contacts exposed to attacker for 20 minutes
- No unauthorized access to Finance systems (blocked before use)
- No data confirmed exfiltrated (but assume attacker has copies of emails)

ROOT CAUSE:
- Weak password (likely from password reuse or data breach)
- No MFA enabled (attacker didn't need second factor)

PREVENTATIVE ACTIONS:
- MFA enabled on Morgan's account
- Impossible Travel CA policy created (Report-only)
- Company-wide MFA enablement initiative planned
- Password hygiene training recommended

FOLLOW-UP TASKS:
- [ ] Notify Morgan of incident and new password requirements
- [ ] Monitor Morgan's account for unusual activity (next 7 days)
- [ ] Check if Morgan's password was in any public breaches (haveibeenpwned.com)
- [ ] Enable MFA for all high-risk roles (Finance, IT, HR)
- [ ] Review Exchange logs to confirm attacker didn't access other mailboxes
```

### Communicate with Morgan

1. **What to tell her:**
   - "Your account showed signs of compromise. We immediately secured it."
   - "We've reset your password. Here's your temporary password (read verbally, not emailed)."
   - "You now have MFA enabled. Set up your authenticator app."
   - "Change your password on next sign-in."
   - "Report any unusual emails sent on your behalf."

2. **What not to tell her:**
   - Don't share technical details unless she asks
   - Don't assign blame ("Your password was weak")
   - Focus on actions and next steps

3. **What to monitor:**
   - Watch for emails from Morgan's account to external addresses
   - Monitor her sign-in logs for Tokyo IPs again
   - Check for new devices or locations added to her account

---

## The Scenario Check

**Incident Response at Contoso:**

An alert arrived at 2:15 PM: Morgan Chen's account was showing impossible travel (NYC → Tokyo, 10 minutes apart). Jordan Kim (IT admin) immediately revoked Morgan's sessions and reset her password. Within 20 minutes, the incident was contained.

Investigation revealed the attacker had access for ~20 minutes, long enough to add themselves as a mailbox delegate and create an email forwarding rule. Jordan removed both, preventing ongoing data theft.

Scope assessment showed only Morgan was directly compromised. No hidden admin accounts were created. The attacker was attempting to escalate to Finance systems but was stopped.

Recovery involved removing the attacker's mailbox access, disabling the forwarding rule, and removing Morgan from unauthorized groups. MFA was enabled on Morgan's account.

Preventative measures included a Conditional Access policy to block or challenge impossible travel attempts company-wide. Finance and IT teams were prioritized for MFA enablement.

The entire incident—from alert to recovery—took ~45 minutes. Documentation was completed by 3 PM. Morgan was notified and re-enabled by 3:30 PM with a new password and MFA enrollment requirement.

---

## Check Your Understanding

- [ ] State the first three actions you would take when notified of account compromise (in order)
- [ ] Explain why revoking sessions is more important than immediately investigating logs
- [ ] Describe the attacker's likely goals when adding themselves as a mailbox delegate
- [ ] List three things you would do differently if the compromise affected a high-privilege account (admin, executive)
- [ ] Create a scenario where the "impossible travel" alert is a false alarm, and describe how you'd handle it

---

## Common Mistakes

**Investigating instead of containing.**
You get a compromise alert. You think, "Let me review logs first." The attacker spends 30 minutes reading emails and creating persistence mechanisms while you investigate. **Revoke sessions immediately, then investigate.**

**Assuming the attacker only used the account once.**
Morgan's account was compromised for 20 minutes. You remove the attacker's immediate access and move on. Weeks later, you discover the attacker's forwarding rule was still active, and they've been receiving emails the whole time. **Always check audit logs to find and undo all attacker changes.**

**Not broadening the scope investigation.**
Only Morgan was compromised. You fix Morgan's account and declare incident closed. Meanwhile, the attacker has also compromised Alex Lee's account (same dark web credential dump). Expand your investigation: Check other users for impossible travel at the same time, unusual group additions, unusual app creations.

**Enabling MFA only on the compromised account.**
Morgan was compromised because she didn't have MFA. You enable it for her. But Alex, River, and Jordan still don't have MFA. When the attacker tries Alex's password (from the same credential dump), they succeed. **Use incidents as a catalyst to strengthen security company-wide, not just for the victim.**

**Not documenting the incident.**
You respond to the incident, fix it, and move on without writing a report. Months later, a similar incident happens. You have no record of the first one, so you don't know this attacker's patterns. **Always document: timeline, what the attacker did, damage, root cause, preventative actions.**

**Trusting the attacker's modifications are gone after you remove them.**
You remove the mailbox delegate. You assume the attacker is locked out. But they also created an OAuth app with `Mail.ReadWrite` permission. They still have access to Morgan's emails through the app. **Search for all attacker persistence mechanisms: delegates, forwarding rules, OAuth apps, hidden accounts, firewall rules.**

---

## What's Next

Unit 6 (Identity Security) is now complete. You've learned to detect (6.1), analyze (6.2), remediate (6.3–6.5), and respond (6.6) to security incidents. You understand access reviews, compliance, and incident response—the three pillars of security operations.

**Unit 7 (Privileged Access Management)** shifts focus from security monitoring to *privilege control*. You'll learn to restrict admin access, require approval for sensitive actions, and audit privileged operations. This is where security becomes proactive: prevent attacks instead of just responding.

---

> 📚 **Entra ID — Zero to Admin** · Unit 6: Identity Security
>
> [← Module 6.5 — Incident Response](../6-5-incident-response/6-5-incident-response.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Unit 7: Privileged Access Management →](../../UNIT%207%20-%20PRIVILEGED%20ACCESS%20MANAGEMENT/7-1-pim-overview/7-1-pim-overview.md)
