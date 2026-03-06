# Reading Sign-In Logs Like a Professional
*Every sign-in is recorded. Knowing what to look for in the logs is the difference between investigation and guessing.*

> 🟡 **Intermediate** · Unit 6, Module 6.2 · ⏱️ 50 minutes

---

## What You Will Learn

- Navigate the sign-in logs and understand the structure
- Identify which fields matter for security investigations
- Filter and search logs by user, app, and location
- Recognize patterns that indicate compromise, failed auth, and legitimate activity
- Read real sign-in logs and extract actionable insights

---

## Why This Matters

*SC-300: Implement authentication and access management — Analyze sign-in activities for compliance and security*

When something goes wrong — user locked out, suspected compromise, suspicious activity — the first place to look is the sign-in logs. The logs record *what happened*: who signed in, from where, to what app, with what result. You've seen sign-in logs throughout earlier modules (Modules 1.1, 3.1, 4.8). This module teaches you to *read them professionally* — understanding what each field means and what patterns indicate problems.

---

## 📋 Sign-In Log Structure

Every sign-in log entry has these key fields:

| Field | What it tells you | Example |
|---|---|---|
| **User** | Who signed in (or tried to) | Alex Lee (alex.lee@contoso.com) |
| **Sign-in time** | When (date and time) | 3/6/2026 2:34:45 PM |
| **Application** | What they signed in to | Office 365 SharePoint Online, Microsoft Teams |
| **Result** | Success or Failure | Success, Interrupted, Failure |
| **Status** | Why it failed (if it did) | Blocked by CA policy, Interrupted by MFA, User not found |
| **IP address** | Where they signed in from | 203.0.113.45 (public IP) |
| **Location** | Where the IP is geographically | United States, New York |
| **Device info** | What device signed in | Browser: Chrome on Windows 10; Device name: LAPTOP-XYZ |
| **Conditional Access** | CA policy applied? | Not applied; Report-only policy applied |
| **MFA result** | Multi-factor authentication | MFA claim in token |
| **Authentication requirement** | What auth method used | Password + Windows Hello, Password + Microsoft Authenticator |

---

## 🔍 What To Look For: Patterns Indicating Problems

**Successful sign-in from expected location and device** ✅ Normal
- User: Alex Lee
- App: Office 365 SharePoint
- Result: Success
- Location: New York (where they always sign in)
- Device: Corporate laptop
- Assessment: Legitimate activity

**Sign-in failure: Blocked by Conditional Access** ⚠️ Expected (policy working)
- User: Alex Lee
- App: Office 365 Exchange
- Result: Interrupted
- Status: Blocked by CA policy (compliant device required)
- Device: Personal phone (not managed by Intune)
- Assessment: Policy enforcement; user needs managed device

**Multiple failed sign-in attempts in short time** 🚨 Suspicious
- User: Morgan Chen
- Multiple entries in 5 minutes
- Result: Failure, Failure, Failure, Success
- Status: Invalid credentials, Invalid credentials, Invalid credentials, Success
- Assessment: Brute-force attack or user trying multiple passwords; attacker likely got in at the end

**Sign-in from impossible location** 🚨 Compromise indicator
- User: Alex Lee
- Sign-in 1: New York at 2:00 PM
- Sign-in 2: Tokyo at 2:10 PM (10 minutes later — physically impossible)
- Result: Both successful
- Assessment: One is legitimate, one is attacker using stolen credentials

**Sign-in from VPN or Tor** ⚠️ Suspicious
- User: Jordan Kim
- Location: IP address traces to VPN provider or Tor exit node
- Result: Success
- Assessment: Could be legitimate (employee using VPN at home) or attacker hiding location

**Successful sign-in at unusual time** ⚠️ Low confidence alone
- User: River Patel
- Sign-in at 3:00 AM (user normally signs in 9-5)
- Result: Success
- Assessment: Could be legitimate (user working late) or compromised account being used by attacker

---

## Lab 6.2.1 — Open and Filter Sign-In Logs

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 1.1 complete; some sign-ins exist in your tenant

1. Navigate to **Monitoring** → **Sign-in logs** (or **Identity** → **Users** → **Sign-in logs**).

2. You'll see a table with columns:
   - User
   - Application
   - Sign-in status
   - Time
   - IP address
   - Location
   - Device name
   - Conditional Access

3. Click a sign-in entry to see full details in a side panel.

4. In the side panel, review:
   - Basic information (user, app, status)
   - Device information (OS, browser, device registered?)
   - Conditional Access (which policies applied?)
   - Sign-in details (IP, location, risk assessment)

5. Use the filter at the top to narrow results:
   - **User** — Search for a specific user
   - **Application** — Filter by app (SharePoint, Teams, etc.)
   - **Status** — Show only failures or successes
   - **Date range** — Search a specific time period

6. Filter by **Status: Failure** to see failed sign-in attempts.

7. Review a few failures and note:
   - Why did they fail? (Status message)
   - What app was the target?
   - What user attempted?

---

## Lab 6.2.2 — Investigate a Suspicious Sign-In Pattern

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 6.2.1 complete

You'll use the logs to investigate a suspicious scenario.

**Scenario:** Alex Lee's account might be compromised. You need to find evidence.

1. Navigate to **Sign-in logs**.

2. Filter by **User: Alex Lee** and **Date range: Last 7 days**.

3. Review all of Alex's sign-ins. Look for:
   - **Sign-ins at unusual times** (middle of the night, weekends, etc.)
   - **Sign-ins from unusual locations** (different countries, unfamiliar IPs)
   - **Multiple failures followed by success** (attacker brute-forcing the password)
   - **Sign-ins from multiple devices simultaneously** (token stolen and used elsewhere)

4. For each suspicious sign-in, click it to view details:
   - IP address → Does it trace to a known company location?
   - Device → Is it a managed/corporate device?
   - Location → Is it where Alex typically is?
   - Authentication → How did they authenticate? (Password only, MFA, etc.)

5. If you find suspicious patterns:
   - Multiple failures from same IP → Brute-force attempt (blocked or unblocked?)
   - Success from new location → Legitimate travel or compromise?
   - MFA not required → Policy gap or user exempt?

6. Document your findings:
   - What pattern did you find?
   - How many suspicious sign-ins?
   - What action would you take? (Block account, force password change, etc.)

---

## Lab 6.2.3 — Use Advanced Filtering to Find Compromised Accounts

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 6.2.2 complete

Identity Protection flags risky signs-in. You can filter logs to see them.

1. Navigate to **Sign-in logs**.

2. Click **Add filters** (or similar button to expand filter options).

3. Look for **Risk level** or **Risk state** filter (if available).

4. Filter by **Risk level: High**.

5. Review all high-risk sign-ins:
   - Which users have them?
   - What apps were targeted?
   - From which locations?

6. Click on a high-risk sign-in to see the risk details:
   - Why was it flagged as high-risk?
   - Impossible travel? Unfamiliar location? Multiple failures?
   - What action did the CA policy take?

7. For each high-risk user, check:
   - Are they a target for attack? (Admin, high-privilege, access to sensitive data?)
   - How many high-risk sign-ins do they have?
   - Should their password be reset?

---

## Lab 6.2.4 — Create a Sign-In Query for Ongoing Monitoring

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 6.2.3 complete

Set up a view you can check regularly.

1. Navigate to **Sign-in logs**.

2. Set a filter that shows interesting activity:
   - **Status: Failure** (to see authentication failures)
   - **Date range: Last 24 hours** (recent activity only)

3. This view shows all failed sign-in attempts from the last day.

4. Bookmark this view or save the filter.

5. Check it daily as part of your security routine:
   - Multiple failures from one user? Investigate.
   - Failures from a legitimate user? Check if they need help.
   - Failures from unfamiliar IPs? Might be attack.

6. Create another view:
   - **Risk level: High** (only high-risk sign-ins)
   - **Status: Success** (they got in despite the risk)

7. This shows successful sign-ins that were risky — investigate why they were allowed and whether remediation is needed.

---

## The Scenario Check

Contoso's sign-in logs are now being reviewed regularly. Jordan Kim (IT admin) checks the logs daily, looking for patterns:
- Alex Lee's account shows multiple sign-ins from NYC (normal) and one from Tokyo (suspicious) — Investigation reveals Alex was traveling
- Morgan Chen has 3 failed sign-ins from the same IP, then 1 success — Investigation shows Morgan changed passwords and forgot the old one
- River Patel has 5 high-risk sign-ins over 2 weeks, all successful — Investigation shows they match CA policies correctly

Monthly audit: Jordan reviews all high-risk sign-ins from the past month, looking for accounts that should have been blocked or investigated further. Patterns emerge that inform security improvements.

---

## Check Your Understanding

- [ ] Navigate to **Sign-in logs** and filter by a specific user — find their last 5 sign-ins
- [ ] Click on one sign-in and identify: user, app, result, location, CA policy applied
- [ ] Find a failed sign-in attempt and explain why it failed
- [ ] Filter for **High risk sign-ins** and review what made them risky
- [ ] Describe what pattern in sign-in logs would indicate a compromised account

---

## Common Mistakes

**Ignoring sign-in log entries that say "Interrupted."**
"Interrupted" doesn't mean failed. It usually means a CA policy or MFA requirement forced additional steps. The user still completed auth. Review Interrupted entries to understand policy effectiveness.

**Assuming all failed sign-ins are attacks.**
A failed sign-in is often just a user with the wrong password. 50 failed sign-ins from one IP in an hour = attack. 1 failed sign-in from a known user device = typo. Context matters.

**Not checking device information.**
A successful sign-in from a new location could be legitimate (user traveling) or compromised (attacker). Check device info: is it a managed corporate device or unknown device? That changes the risk assessment.

**Overlooking time-based patterns.**
A user normally signs in 9-5 M-F. A sign-in at 2 AM on Sunday is suspicious. Always check when sign-ins happen relative to the user's normal patterns.

**Not acting on high-risk sign-ins.**
Entra ID flags a sign-in as high-risk. You see it in the logs. You do nothing. If the risk policy is enabled, the user was already challenged (MFA) or blocked. But if policies aren't strict enough, the high-risk user might have been allowed in without remediation. Check what risk status means and whether action is needed.

---

## What's Next

Module 6.3 teaches **Risky Users and Remediation** — when Entra ID flags an entire user account as high-risk (not just one sign-in), what that means, how to investigate, and how to remediate (reset password, force re-authentication, etc.).

---

> 📚 **Entra ID — Zero to Admin** · Unit 6: Identity Security
>
> [← Module 6.1 — Identity Protection](../6-1-identity-protection-risk-detection/6-1-identity-protection-risk-detection.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 6.3 — Risky Users and Remediation →](../6-3-risky-users-remediation/6-3-risky-users-remediation.md)
