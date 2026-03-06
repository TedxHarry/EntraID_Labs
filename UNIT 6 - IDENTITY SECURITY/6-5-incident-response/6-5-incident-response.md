# Incident Response — Compromised Accounts
*Your account is actively being attacked. Here's the step-by-step playbook to contain, investigate, and recover.*

> 🔴 **Advanced** · Unit 6, Module 6.5

---

## What You Will Learn

- Recognize signs of active account compromise
- Execute immediate containment steps (revoke sessions, reset password, disable account)
- Investigate what the attacker accessed and changed
- Recover the account for legitimate use
- Prevent recompromise with stronger authentication and monitoring

---

## Why This Matters

*SC-300: Implement authentication and access management — Respond to security incidents involving compromised user accounts*

Modules 6.1–6.4 taught you to detect compromise. This module teaches you to *respond* when detection becomes reality. An attacker has broken into Alex Lee's account and is actively using it. You have minutes, not hours, to contain the damage. This is the playbook Entra ID admins follow in real incidents: stop the attacker immediately, understand what they did, fix the account, and prevent it from happening again.

---

## 🚨 The Incident: Account Actively Compromised

**Scenario:** It's 2 PM on a Tuesday. Your security team alerts you: "Alex Lee's account is showing impossible travel — sign-in from New York at 1 PM, Tokyo at 1:15 PM. Then 10 failed password changes in 5 minutes. Her mailbox was accessed from a strange IP. Someone has her account."

**Your response window:** Minutes. The attacker is *currently* using the account. Every minute they have access, they can:
- Read all emails (steal customer data, steal IP)
- Forward all future emails to themselves
- Modify her access (add themselves as delegate)
- Change passwords on other systems (lateral movement)
- Delete evidence (calendar invites, email threads)

This module walks you through the response playbook.

---

## 📋 The Incident Response Playbook: Four Phases

### Phase 1: Immediate Containment (0–2 minutes)
**Goal:** Stop the attacker *right now*. Disable the account so they can no longer access anything.

- Revoke all active sessions (sign out the attacker everywhere)
- Reset the password (the attacker's password becomes invalid)
- Optionally: Disable the account entirely (nuclear option for high-risk accounts)
- Optionally: Block sign-in (Conditional Access policy block)

### Phase 2: Investigation (2–15 minutes)
**Goal:** Understand what the attacker did, how long they were in, and what they accessed.

- Review sign-in logs (where did they sign in from? What device? How many times?)
- Review Entra ID audit logs (what did they change? Add admins? Create app registrations?)
- Review Exchange logs (what emails did they read/send/delete?)
- Review Teams logs (what channels did they access?)
- Identify scope of compromise (one mailbox or widespread access?)

### Phase 3: Recovery (15–60 minutes)
**Goal:** Restore the account to legitimate use with proper authentication.

- Reset password with a temporary secure value
- Enable stronger authentication (MFA, passwordless)
- Review and restore any deleted items
- Monitor the account for attacker reentry attempts
- Communicate with the user: new password, what happened, what to watch for

### Phase 4: Prevention (1–24 hours)
**Goal:** Prevent this from happening again to this user or others.

- Review how the account was compromised (password stolen? Phishing? MFA bypass?)
- Strengthen authentication org-wide
- Enable additional monitoring and alerting
- Update incident response playbooks

---

## Lab 6.5.1 — Revoke All Sessions for a Compromised Account

**Tools:** Entra admin center
**Prerequisite:** Module 6.1 complete; a test user account exists

> ⚠️ **Real Incident Awareness:** In production, revoking sessions is the first action you take. It immediately signs out anyone using the account—including the attacker.

1. Navigate to **Users** → **All users**.

2. Search for and click on **Alex Lee** (the compromised account).

3. Click **Sign-out sessions** (or **Revoke sessions** depending on portal version).

   - You'll see a confirmation: "Sign out user from all sessions?"
   - Click **Yes**.

4. **What just happened:**
   - Every active session for Alex is terminated
   - Her browser tabs that were signed in now require re-authentication
   - The attacker's sessions (on their machine) also terminate
   - She's signed out everywhere

5. In the background, Entra ID is now revoking:
   - Browser session tokens
   - Mobile app session tokens
   - Refresh tokens (attackers often use these for long-term access)

6. **Verification:** Alex tries to access Teams, Outlook, etc. She's signed out. She must sign in again (with the old password, which is still valid — we'll reset it next).

---

## Lab 6.5.2 — Reset Password for Compromised Account

**Tools:** Entra admin center
**Prerequisite:** Lab 6.5.1 complete

Now that sessions are revoked, reset the password so the attacker's password becomes invalid.

1. On Alex Lee's user profile, find **Password** section (or click **Reset password**).

2. Click **Reset password**.

   - You'll see a prompt: "Generate a temporary password?"
   - Click **Yes** (Entra ID generates a secure random password)

3. **Entra ID displays the temporary password:** (example) `TempP@ssw0rd123!`
   - Copy this password
   - **Never put it in email** — the attacker might intercept email
   - Call Alex directly and read it to her verbally, or use a password manager, or SMS if available

4. Check the box: **User must change password on next sign-in** (if available).

   - This forces Alex to choose her own password immediately upon sign-in
   - The attacker can't sign in with the temporary password

5. Click **Reset**.

6. **What just happened:**
   - The attacker's password (whatever they stole) is now invalid
   - Even if the attacker still has the stolen password, it won't work
   - Sessions were revoked, and password was reset—the attacker is locked out

---

## Lab 6.5.3 — Review Sign-In Logs to Understand the Attack

**Tools:** Entra admin center
**Prerequisite:** Lab 6.5.2 complete

Investigate what the attacker did while they had access.

1. On Alex Lee's user profile, click **Sign-in logs** (or navigate to **Monitoring** → **Sign-in logs** and filter for Alex).

2. Review the suspicious sign-ins:
   - **Location:** Where was the attacker signing in from? (IP address, country, city)
   - **Device info:** What device/browser did they use?
   - **Success or failure:** Did the sign-in succeed, or did they try and fail?
   - **Time:** When did they sign in? Is it outside normal working hours?

3. Look for patterns:
   - **Impossible travel:** Multiple sign-ins from different countries in a short time
   - **Multiple failures:** The attacker tried the password many times (brute force)
   - **Unusual devices:** Sign-ins from machines Alex doesn't normally use
   - **Off-hours access:** Sign-ins at 3 AM when Alex normally sleeps

4. **Example timeline you might see:**
   - 9:00 AM: Alex signs in from office (New York) — Normal ✓
   - 1:15 PM: Sign-in from Tokyo IP, unknown device — Suspicious ⚠️
   - 1:20 PM: Attempted sign-in from Tokyo, failed — Suspicious ⚠️
   - 1:25 PM: Sign-in from New York, unknown device — Suspicious ⚠️
   - 2:00 PM: Your admin action: Revoked sessions ← **Containment point**

5. **Action:** Document the timeline:
   - When did the compromise start?
   - How long did the attacker have access?
   - What was their geographic location(s)?
   - What devices did they use?
   - Did they succeed in signing in, or did MFA block them?

---

## Lab 6.5.4 — Review Audit Logs for Account Modifications

**Tools:** Entra admin center
**Prerequisite:** Lab 6.5.3 complete

Beyond sign-ins, check what the attacker *did* while they had access.

1. Navigate to **Audit logs** (or **Monitoring** → **Audit logs**).

2. Filter:
   - **User:** Alex Lee
   - **Activity:** All activities
   - **Date range:** Last 24 hours (or the timeframe of the suspected compromise)

3. Review for suspicious changes. Look for:

   **User account modifications:**
   - Password changed (should be your admin reset, not an attacker reset)
   - Email address changed
   - Display name changed
   - Manager changed

   **Access grants:**
   - User added to high-privilege groups (Admins, Security Admins)
   - User added to application with sensitive access
   - Application permissions granted

   **Delegated access:**
   - Mailbox delegate added (attacker adding themselves to read all emails)
   - Manager changed
   - Team member added to sensitive groups

   **Data access:**
   - Files deleted
   - Shared items modified

4. **Example finding:** The audit log shows:
   - 1:30 PM: Mailbox delegate added (attacker added someone@attacker.com as a mailbox delegate)
   - This is a **red flag** — attacker set up persistent access to Alex's email
   - **Action:** Remove the delegate immediately (next lab)

5. If you find suspicious changes, document:
   - What was changed?
   - When was it changed?
   - What is the impact?
   - Can it be undone (restore from backup)?

---

## Lab 6.5.5 — Undo Attacker Changes (Mailbox Delegates)

**Tools:** Outlook/Exchange admin center
**Prerequisite:** Lab 6.5.4 complete; you found suspicious delegates

If the attacker added themselves as a mailbox delegate (to read all emails forever), remove them.

1. Navigate to **Exchange admin center** (or **Outlook** → **Settings** → **Forwarding**)

2. Find **Mailbox delegates** or **Mailbox forwarding** for Alex Lee.

3. If you see unauthorized delegates (example: attacker@attackerdomain.com):
   - Click **Remove** or **Delete** next to them
   - Confirm removal

4. **What this does:**
   - The attacker can no longer read Alex's emails
   - The attacker's persistent access is removed
   - The attacker must have the password to access again (which was just reset)

5. Check for **email forwarding rules** as well:
   - The attacker might have created a rule: "Forward all my emails to attacker@attackerdomain.com"
   - If found, delete the rule

6. **Verification:** The attacker's mailbox delegate is gone. Their access to Alex's emails is revoked.

---

## Lab 6.5.6 — Strengthen Authentication (Enable MFA and Passwordless)

**Tools:** Entra admin center
**Prerequisite:** Lab 6.5.5 complete; account is now contained

Now that the account is secured, strengthen it to prevent recompromise.

1. Navigate to **Users** → **All users** → **Alex Lee**.

2. Click **Authentication methods** (or **Multi-factor authentication**).

3. Ensure Alex has **strong authentication** enabled:
   - **Option A: MFA (standard)** — Require password + phone/authenticator app
     - Enable "Require multifactor authentication"
     - Alex must complete MFA (receive code on phone) to sign in

   - **Option B: Passwordless (modern)** — No password at all, use Windows Hello or authenticator app
     - More secure, eliminates password-based attacks
     - Requires user to enroll

4. For immediate security, enable MFA:
   - Toggle **Require multifactor authentication** to **Yes**
   - Click **Save**

5. **What this means:**
   - Even if the attacker gets a password later, they need Alex's phone to sign in
   - Passwordless (later, optional) removes passwords entirely

6. **Communicate with Alex:**
   - New password (temporary, must change on sign-in)
   - MFA requirement (set up authenticator app)
   - What happened (high-level explanation: account was compromised, security actions taken)
   - What to watch for (unusual emails sent on her behalf, unexpected account activity)

---

## Lab 6.5.7 — Create an Incident Response Policy (Automation)

**Tools:** Entra admin center
**Prerequisite:** Module 6.1 complete; Identity Protection is enabled

Automate some containment actions so they happen instantly when compromise is detected.

1. Navigate to **Protection** → **Identity Protection** → **Risk policies** → **User risk policy**.

2. Set **Enforce policy** to **On**.

3. Configure:
   - **Medium and above:** Require password change (user must reset password when account shows compromise signs)
   - **High:** Require password change (even stricter for high-risk accounts)

4. Click **Save**.

5. This policy now automatically responds to detected compromises:
   - If Identity Protection detects high user risk (leaked credentials on dark web), the policy forces a password change
   - Users cannot bypass it — password change is mandatory
   - No admin action needed; it's automatic

6. **Result:** Compromised accounts are partially remediated without admin involvement, buying time for investigation.

---

## The Scenario Check

Contoso's Alex Lee's account was compromised. Here's how the incident unfolded:

**T+0:00 — Detection:** Security team alerts Jordan Kim (IT admin): "Alex's account shows impossible travel and unusual access."

**T+0:01 — Immediate containment:** Jordan navigates to Alex's user profile and revokes all sessions. The attacker is signed out everywhere.

**T+0:02 — Password reset:** Jordan resets Alex's password. The stolen password (attacker had it) is now invalid.

**T+0:05 — Investigation begins:** Jordan reviews sign-in logs. Alex signed in from NYC normally. Then an attacker signed in from Tokyo IP at 1:15 PM, bypassed MFA somehow, and accessed the account for 45 minutes.

**T+0:15 — Damage assessment:** Jordan reviews audit logs. The attacker added attacker@attackerdomain.com as a mailbox delegate. They accessed Teams channels with customer data.

**T+0:20 — Undo damage:** Jordan removes the attacker's mailbox delegate. Exchange audit logs confirm the delegate is gone.

**T+0:30 — Strengthen security:** Jordan enables MFA on Alex's account. Even if the attacker gets a password later, they need her phone to sign in.

**T+0:35 — Communicate:** Jordan calls Alex. Explains what happened, gives her the temporary password verbally (not email), tells her to change it immediately and enroll in MFA.

**T+1:00 — Follow-up:** Jordan creates a task to investigate: How did the attacker get the password? (Phishing? Password reuse? Data breach?) and to enable passwordless authentication org-wide to prevent similar incidents.

---

## Check Your Understanding

- [ ] Explain why revoking sessions is the first step in incident response (not resetting password)
- [ ] List three things you would look for in sign-in logs when investigating a compromised account
- [ ] Describe what a mailbox delegate does and why an attacker would add one
- [ ] State what happens to an account after you reset its password
- [ ] Explain how MFA prevents recompromise even if the password is stolen again

---

## Common Mistakes

**Waiting to confirm the compromise before taking action.**
You get an alert: "Suspicious sign-ins detected." You think, "Let me investigate first." Meanwhile, the attacker is actively reading emails, creating rules, changing passwords. Revoke sessions immediately; investigate *after* containment. You can restore legitimate access if it was a false alarm. You can't undo damage the attacker did while you were investigating.

**Resetting the password but forgetting to revoke sessions.**
You reset Alex's password, thinking the attacker is locked out. But the attacker still has valid session tokens (from before the reset). They continue accessing the account without needing the password. **Always revoke sessions first, then reset password.**

**Assuming the attacker accessed only the compromised account.**
Alex's account was compromised. You secure her account and move on. But the attacker might have lateral movement — accessing other accounts, changing permissions, creating hidden admins. After containing the primary account, widen the investigation: Check if other users had impossible travel at the same time, if new admins were created, if service principals were given broad permissions.

**Not reviewing audit logs for attacker modifications.**
You reset the password and move on. Weeks later, you discover the attacker had added a mailbox delegate and set up an email forwarding rule. Customer data was slowly exfiltrated. Always review audit logs to understand *what the attacker did*, not just *how they signed in*.

**Enabling MFA on the user's existing device.**
You enable MFA on Alex's account. She has her phone. But if the attacker also has her phone (they stole it), they can intercept MFA codes. If possible, issue a new phone or hardware security key. If not, change her phone number to a number the attacker doesn't know.

---

## What's Next

Module 6.6 (Breakfix Lab) is a hands-on simulation where you respond to a real-looking incident from start to finish: detect compromise, contain it, investigate, recover, and strengthen security. You'll practice the full playbook learned in this module.

---

> 📚 **Entra ID — Zero to Admin** · Unit 6: Identity Security
>
> [← Module 6.4 — Access Reviews](../6-4-access-reviews/6-4-access-reviews.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 6.6 — Breakfix: Incident Response Lab →](../6-6-incident-response-breakfix/6-6-incident-response-breakfix.md)
