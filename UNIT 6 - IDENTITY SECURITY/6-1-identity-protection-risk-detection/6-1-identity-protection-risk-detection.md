# Identity Protection and Risk Detection
*Entra ID watches every sign-in for signs of compromise. Here's what it's looking for and what you can do about it.*

> 🟡 **Intermediate** · Unit 6, Module 6.1

---

## What You Will Learn

- Explain what Identity Protection is and why it matters
- Describe the two types of risk: sign-in risk and user risk
- Identify common risk detections (impossible travel, unfamiliar locations, suspicious activity)
- Configure Identity Protection policies to respond to risky sign-ins
- Use the Identity Protection dashboard to view risk detections

---

## Why This Matters

*SC-300: Implement authentication and access management — Plan and implement risk-based conditional access policies*

Units 1–5 taught you to build identity infrastructure and control access. Unit 6 teaches you to defend it. Identity Protection is Entra ID's attack detection system. It watches for account compromise (passwords stolen, tokens hijacked, accounts taken over) and alerts admins. Without it, you don't know your accounts are compromised until data is stolen. With it, you can block attackers in real time.

---

## 🔍 What Is Identity Protection

**Identity Protection** is Entra ID's security monitoring service that:

1. **Detects risk** — Watches every sign-in for suspicious patterns
2. **Calculates risk level** — Low, medium, high (based on how suspicious it is)
3. **Reports findings** — Shows admins what it found
4. **Enables response** — Blocks risky sign-ins or forces additional authentication

Think of it as a security camera for sign-in events — it watches, records, and alerts you when something looks wrong.

---

## ⚠️ Two Types of Risk

**Sign-in Risk** — Indicates the current sign-in is suspicious

| Risk Level | What it means | Example |
|---|---|---|
| **Low** | Sign-in has minor suspicious signals | User signed in from a new device (first time ever) |
| **Medium** | Multiple suspicious signals | Sign-in from unfamiliar location + unusual time |
| **High** | Strong evidence of compromise | Impossible travel (user in NYC, then Tokyo in 10 minutes) |

**User Risk** — Indicates the user's account might be compromised

| Risk Level | What it means | Why it matters |
|---|---|---|
| **Low** | Minor indicators | Account hasn't shown compromise patterns |
| **Medium** | Account showing compromise patterns | Someone might be testing the password |
| **High** | Strong evidence account is compromised | Password/tokens were stolen and used |

---

## 🚨 Common Risk Detections

**Impossible Travel**
- User signs in from NYC at 2pm
- User signs in from Tokyo at 3pm (physically impossible in 1 hour)
- Detection: High sign-in risk
- Reality: User's password stolen; attacker using it from another country

**Unfamiliar location**
- User always signs in from office (New York)
- Today they sign in from Bangkok (first time)
- Detection: Medium sign-in risk
- Reality: User on vacation, or password stolen

**Suspicious activity**
- Multiple failed sign-in attempts in short time
- Sign-in from VPN or proxy (attacker hiding location)
- Sign-in from TOR browser (anonymity network)
- Detection: Medium to High sign-in risk
- Reality: Someone brute-forcing the password, or attacker being cautious

**Leaked credentials**
- Entra ID knows user's password was leaked on the dark web
- Multiple accounts using the same leaked password
- Detection: High user risk
- Reality: Password was in a data breach; attacker likely has it

**Anomalous token**
- Token shows characteristics of malware-generated tokens
- Token has unusual claims or signatures
- Detection: Medium to High sign-in risk
- Reality: Malware created a fake token, or token was hijacked

---

## Lab 6.1.1 — Enable Identity Protection

**Tools:** Entra admin center
**Prerequisite:** Module 1.1 (tenant exists)

> 🔑 **Licensing:** Identity Protection requires **Entra ID P2** license. Your developer tenant includes this. In production, verify you have P2 before enabling.

1. Navigate to **Protection** → **Identity Protection** → **Overview**.

2. You'll see the Identity Protection dashboard with three sections:
   - **Risky users** — Accounts showing signs of compromise
   - **Risky sign-ins** — Individual sign-ins that looked suspicious
   - **Risk detections** — Specific suspicious patterns detected

3. All sections are likely empty (new tenant). This is normal.

4. On the left menu, click **Risk policies**.

5. You'll see two policy types:
   - **User risk policy** — Respond to compromised accounts
   - **Sign-in risk policy** — Respond to suspicious sign-ins

6. Both are currently disabled (Off). Enable them:
   - Click **User risk policy**
   - Set **Enforce policy** to **On**
   - Set **Medium and above** to **Require password change** (if risk is medium or higher, force the user to change their password)
   - Click **Save**

7. Repeat for **Sign-in risk policy**:
   - Click **Sign-in risk policy**
   - Set **Enforce policy** to **On**
   - Set **Medium and above** to **Require multifactor authentication** (if risk is medium or higher, require MFA even if user normally doesn't use it)
   - Click **Save**

Now Identity Protection is active and will respond to risky sign-ins.

---

## Lab 6.1.2 — Explore the Identity Protection Dashboard

**Tools:** Entra admin center
**Prerequisite:** Lab 6.1.1 complete

1. Navigate to **Protection** → **Identity Protection** → **Overview**.

2. Review the three sections:

   **Risky users:**
   - Shows users whose accounts are flagged as compromised
   - If empty, no users currently show high risk
   - Click on a user to see risk details

   **Risky sign-ins:**
   - Shows individual sign-in attempts that were suspicious
   - If empty, no suspicious sign-ins detected yet
   - Click on a sign-in to see what made it suspicious

   **Risk detections:**
   - Shows specific risk events (impossible travel, leaked credentials, etc.)
   - Categorized by risk level and detection type
   - Historical view — shows what was detected over time

3. On the left menu, explore:
   - **Risky users** — Full list of flagged users with timeline
   - **Risky sign-ins** — Full list of suspicious sign-in attempts
   - **Risk detections** — All risk events with details

4. Currently, these are likely empty. In a production tenant with real sign-ins, you'd see activity here.

---

## Lab 6.1.3 — Simulate a Risky Sign-In (Understanding Risk Detection)

**Tools:** InPrivate browser, multiple locations
**Prerequisite:** Lab 6.1.2 complete

You'll simulate what a risky sign-in looks like from an admin perspective.

**Scenario:** Alex Lee signs in from an unusual location.

1. Open an **InPrivate** browser window.

2. Navigate to `myapps.microsoft.com`.

3. Sign in as **alex.lee@yourdomain.onmicrosoft.com**.

4. Complete MFA if prompted.

5. You're signed in. Close the window.

6. Now, in your admin browser, navigate to **Protection** → **Identity Protection** → **Risky sign-ins**.

7. Look for Alex's sign-in. You might see it listed with details:
   - User: Alex Lee
   - Risk level: Low or Medium (depending on Entra ID's detection)
   - Risk details: What made it suspicious (new device, new location, etc.)

8. Click on Alex's sign-in to see full details:
   - Location (IP address, country, city)
   - Device info (browser, OS, whether device is managed)
   - Risk factors detected
   - Risk level justification

9. This is what admins see when investigating suspicious sign-ins.

---

## Lab 6.1.4 — Test Risk-Based Conditional Access

**Tools:** Entra admin center
**Prerequisite:** Lab 6.1.3 complete; Module 4.8 (CA policies) understood

Identity Protection integrates with Conditional Access. You can create policies that respond to detected risk.

1. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

2. **Name:** `Block High Risk Sign-ins`

3. **Assignments:**
   - **Users and groups:** Select **All users**
   - **Target resources:** Select **All cloud apps**

4. **Conditions:**
   - Click **Sign-in risk** → Toggle **Configured**
   - Select **High**

5. **Access control** (Grant):
   - Click **Block access**

6. **State:** **Report-only** (test before enforcing)

7. Click **Create**.

This policy says: "If a sign-in is flagged as High risk, block it (in Report-Only mode for now)."

When switched to Enforce, any sign-in Entra ID flags as High risk will be blocked.

> 💡 **Tip:** Remember to exclude your break-glass emergency account from this policy (using the `CA-Exclusion-BreakGlass` group from Module 4.1).

---

## The Scenario Check

Contoso's Identity Protection is now enabled. Risk policies are active: medium-and-above sign-in risk requires MFA; medium-and-above user risk requires password change.

Alex Lee signed in from an unusual location. Entra ID detected this, flagged the sign-in as medium risk, and required MFA. Alex completed MFA and was allowed in — but the sign-in is now recorded in **Risky sign-ins** for Jordan Kim (IT admin) to review.

A Conditional Access policy blocks high-risk sign-ins in Report-Only mode, documenting what would be blocked when switched to Enforce.

If an account later shows signs of compromise (leaked credentials detected on the dark web), that account is flagged as high user risk. The user risk policy forces a password change, protecting the account from unauthorized use.

---

## Check Your Understanding

- [ ] Navigate to **Identity Protection** → **Overview** and verify it's enabled
- [ ] Describe the difference between sign-in risk and user risk
- [ ] Find a risk detection in the dashboard and explain what made it risky
- [ ] State what happens when a sign-in is flagged as High risk (with policies enabled)
- [ ] Explain why "impossible travel" is a reliable indicator of compromise

---

## Common Mistakes

**Not enabling Identity Protection in P2 tenants.**
You have Entra ID P2 but never enable Identity Protection. Compromised accounts go undetected. Enable it as part of your security baseline.

**Setting risk policies to Report-Only and forgetting to enable them.**
You configure a user risk policy in Report-Only mode to test. Weeks later, you find out it's still in Report-Only — nobody's been enforced. Set it to **On** after testing.

**Blocking all High risk sign-ins without exceptions.**
Your break-glass emergency account is not excluded from a "Block High risk" policy. During an incident, you try to sign in with the break-glass account and get blocked because Entra ID flagged it as high risk (it's being used from an unusual location). Always exclude break-glass accounts.

**Trusting risk scores blindly.**
Entra ID flags a sign-in as High risk → you assume it's definitely an attack. Sometimes it's a false positive (user traveled, unusual device, etc.). Use risk flags as *alerts*, not *certainties*. Investigate before taking action.

---

## What's Next

Module 6.2 teaches you to **read sign-in logs professionally** — how to use the logs Entra ID generates to investigate security incidents, understand user behavior, and diagnose authentication failures. The logs are where the evidence is; this module teaches you to read them.

---

> 📚 **Entra ID — Zero to Admin** · Unit 6: Identity Security
>
> [← Module 5.9 — Consent Settings](../../UNIT%205%20-%20APPLICATIONS%20AND%20SSO/5-9-consent-workflow/5-9-consent-workflow.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 6.2 — Reading Sign-In Logs →](../6-2-reading-signin-logs/6-2-reading-signin-logs.md)
