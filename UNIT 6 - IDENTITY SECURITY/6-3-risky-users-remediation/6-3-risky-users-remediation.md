# Risky Users and Remediation
*When an account is flagged as compromised, you have minutes to act before the attacker steals everything.*

> 🟡 **Intermediate** · Unit 6, Module 6.3

---

## What You Will Learn

- Understand what makes an account "risky" (user risk vs sign-in risk)
- Review risky users in the Identity Protection dashboard
- Investigate whether a risky user account is actually compromised
- Remediate a compromised account (password reset, revoke sessions, etc.)
- Prevent future compromise with policies and MFA

---

## Why This Matters

*SC-300: Implement authentication and access management — Respond to compromised accounts*

When Entra ID detects a compromised account (leaked credentials, suspicious patterns, etc.), seconds count. If you don't remediate quickly, the attacker steals data, impersonates the user, escalates privileges. This module teaches you the immediate actions: disable the account, force password change, revoke sessions, and investigate what happened.

---

## 🚨 What "Risky User" Means

**User Risk** is different from **Sign-in Risk:**

| | Sign-in Risk | User Risk |
|---|---|---|
| **Scope** | One specific sign-in event | The entire user account |
| **Detection** | One suspicious sign-in | Multiple suspicious patterns or leaked credentials |
| **Example** | User signs in from impossible location | User's password was leaked on dark web; account used by attacker multiple times |
| **Urgency** | High (immediate response) | Critical (immediate response required) |
| **Response** | Challenge with MFA or block | Force password change, revoke all sessions, investigate |

When Entra ID flags a **user as risky**, it means the account itself is likely compromised — not just this sign-in, but the account's security overall.

---

## 🔍 Common Reasons an Account is Flagged as Risky

**Leaked credentials**
- User's password found in a data breach (dark web, leaked password database)
- Entra ID knows the password is public
- Attacker likely has it

**Multiple suspicious sign-ins**
- Account used from 5 different countries in one hour
- Multiple failed sign-in attempts followed by success
- Sign-ins at 3 AM when user normally sleeps

**Token-stealing malware**
- Malware on the user's device steals tokens
- Multiple sign-ins from the malware-infected device
- Pattern indicates token hijacking

**Compromised device**
- Device shows malware indicators
- Device has no antivirus, unpatched OS, etc.
- Account used from that device looks suspicious

---

## Lab 6.3.1 — Review Risky Users Dashboard

**Tools:** Entra admin center
**Prerequisite:** Module 6.1 complete

1. Navigate to **Protection** → **Identity Protection** → **Risky users**.

2. You'll see a list of users Entra ID has flagged as risky (likely empty in a fresh tenant).

3. For each risky user listed, you see:
   - **User name**
   - **Risk level** (Low, Medium, High)
   - **Risk state** (Active, Resolved, Dismissed)
   - **Last update** (when the risk was detected)

4. Click on a risky user to see details:
   - **Risk history** — Timeline of when risk was detected
   - **Risk events** — Specific events that triggered the flag (leaked credentials, suspicious sign-in, etc.)
   - **Remediation status** — Has the risk been addressed?

5. Typical risk events you might see:
   - "Leaked credentials" (password found in breach)
   - "Anomalous sign-in" (unusual location/time)
   - "Suspicious activity" (malware patterns)

---

## Lab 6.3.2 — Investigate a Risky User

**Tools:** Entra admin center
**Prerequisite:** Lab 6.3.1 complete; you have a risky user to investigate

**Scenario:** Entra ID flagged River Patel as risky. You need to determine if the account is actually compromised or if it's a false positive.

1. Navigate to **Risky users** and click on **River Patel** (or any risky user).

2. Review the risk details:
   - **Risk detection reason** — Why was it flagged? (leaked credentials, anomalous sign-in, etc.)
   - **Risk level** — How confident is Entra ID? (Low = might be false positive, High = very likely compromised)

3. Cross-reference with sign-in logs:
   - Navigate to **Monitoring** → **Sign-in logs**
   - Filter by **User: River Patel**, **Last 24 hours**
   - Look for the sign-ins that triggered the risk flag
   - Do they look legitimate or suspicious?

4. Ask yourself:
   - **Leaked credentials:** Is River's password known to be in a breach? (Entra ID would tell you)
   - **Anomalous sign-in:** Where was River? Could they legitimately be in that location?
   - **Suspicious activity:** Is this activity consistent with River's normal patterns?

5. Determine: **Legitimate or Compromised?**
   - **Legitimate:** False positive. River traveled, used new device, changed habits. No actual compromise.
   - **Compromised:** Attacker has River's credentials. Account needs immediate remediation.

---

## Lab 6.3.3 — Remediate a Compromised Account

**Tools:** Entra admin center, PowerShell (optional)
**Prerequisite:** Lab 6.3.2 complete; you've identified an account as compromised

**Remediation Steps (in order of urgency):**

**Step 1: Revoke all sessions immediately**

1. Navigate to **Identity** → **Users** → **All users** → Click the compromised user (e.g., River Patel).

2. Click **Sign-out all sessions** (or similar button).

3. This immediately logs the user out of all apps and browsers. Any attacker currently using the account is disconnected.

4. Confirm the action.

**Step 2: Reset the password**

5. On the user's profile, click **Reset password**.

6. A temporary password is generated. Copy it (you'll give it to the user).

7. Note: The user must change this temp password on next sign-in.

**Step 3: Force MFA enrollment**

8. On the user's profile, click **Authentication methods** (or **MFA settings**).

9. If the user has no MFA methods, they must register one.

10. Set **Require re-register MFA** to **Yes** (forces them to set up MFA again on next sign-in).

**Step 4: (Optional) Disable the account temporarily**

11. If you suspect active attacker use right now:
    - Go to **User** → **Account status**
    - Toggle to **Disabled**
    - This prevents any sign-in until you enable it again

12. Investigate further, then enable the account once you're confident it's secure.

**Step 5: Document and communicate**

13. Document what happened:
    - Why was the account flagged as risky?
    - What remediation actions were taken?
    - When was it done?
    - Who did it (you)?

14. Contact the user:
    - Inform them their account was compromised
    - Provide the temporary password
    - Instruct them to change it on next sign-in
    - Ask them to review recent activity and report any suspicious actions

---

## Lab 6.3.4 — Dismiss or Resolve Risk (After Remediation)

**Tools:** Entra admin center
**Prerequisite:** Lab 6.3.3 complete; remediation done

After you've remediated the account, dismiss or resolve the risk flag.

1. Navigate to **Risky users** and click on the remediated user.

2. At the top, you'll see a button: **Dismiss user risk** or **Resolve risk**.

3. Click it. The risk state changes from **Active** to **Dismissed** or **Resolved**.

4. This tells Entra ID and other admins that you've investigated and addressed the risk.

5. Future sign-ins from this user will be monitored normally again (not flagged automatically as risky).

> ⚠️ **Warning:** Only dismiss if you're confident the account is secure. Dismissing without remediation leaves the account compromised.

---

## Lab 6.3.5 — Create a User Risk Remediation Policy

**Tools:** Entra admin center
**Prerequisite:** Module 6.1 complete

Set up a policy that automatically responds to user risk.

1. Navigate to **Protection** → **Identity Protection** → **Risk policies** → **User risk policy**.

2. **Enforce policy:** Toggle to **On** (if not already).

3. Set **Users with high risk** to **Require password change**.

4. This means: If a user is flagged as high-risk, they must change their password before proceeding.

5. **Save**.

Now any user flagged as high-risk will be forced to change their password on the next sign-in, even if they're not currently signing in. This proactively locks out attackers.

---

## The Scenario Check

Contoso has a risky user alert: Alex Lee's password was found in a leaked database. Entra ID flagged her account as high-risk.

Jordan Kim (IT admin) immediately:
1. Revoked all of Alex's sessions (logged her out everywhere)
2. Reset her password to a temporary one
3. Forced MFA re-enrollment
4. Contacted Alex and provided the temporary password
5. Investigated the sign-in logs — found no suspicious activity (the password was leaked but not used by an attacker)
6. Confirmed the account was secure and dismissed the risk flag

Alex changed her temporary password on next sign-in and completed new MFA setup. Her account is now secure.

The user risk remediation policy ensures future high-risk users are automatically forced to change their password, preventing the attacker from using a stolen password.

---

## Check Your Understanding

- [ ] Navigate to **Risky users** and identify any flagged users
- [ ] For a risky user, state what risk event triggered the flag
- [ ] Explain the difference between dismissing a risk and resolving it
- [ ] Describe the three remediation steps you'd take for a compromised account (in order)
- [ ] State what happens when you click **Sign-out all sessions** for a compromised account

---

## Common Mistakes

**Not revoking sessions when remediating.**
You reset a compromised user's password. The attacker is still logged in to Teams, SharePoint, etc., using the old session. They continue stealing data. Always revoke all sessions first.

**Dismissing risk without actually investigating.**
Entra ID flags an account as high-risk. You see the flag and immediately dismiss it without reviewing sign-ins or understanding why it was flagged. Attacker continues using the account. Always investigate before dismissing.

**Waiting to remediate a high-risk account.**
You see a user flagged as high-risk at 2pm. You plan to remediate tomorrow morning. By then, the attacker has reset the password, enabled MFA with their own phone, and locked out the legitimate user. Remediate immediately, not later.

**Not notifying the user of remediation.**
You reset the user's password due to compromise. You don't tell them. They try to sign in with their old password and get locked out. They call the help desk frustrated. Always communicate — "Your account was compromised, we reset your password, here's the temporary one."

**Using the same temporary password for multiple remediation events.**
You reset 5 compromised accounts and use the same temp password for all of them. If one user sees another's temp password, they can access that account. Use unique passwords for each user.

---

## What's Next

Module 6.4 teaches **Access Reviews — Who Really Needs Access?** You'll learn to audit user access: which users have access to which apps/resources, remove access from people who no longer need it, and prove compliance by reviewing access regularly. This prevents accumulation of excessive permissions over time.

---

> 📚 **Entra ID — Zero to Admin** · Unit 6: Identity Security
>
> [← Module 6.2 — Reading Sign-In Logs](../6-2-reading-signin-logs/6-2-reading-signin-logs.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 6.4 — Access Reviews →](../6-4-access-reviews/6-4-access-reviews.md)
