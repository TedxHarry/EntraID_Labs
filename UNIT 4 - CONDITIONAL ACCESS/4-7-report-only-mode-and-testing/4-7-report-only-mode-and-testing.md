# Report-Only Mode and Testing Safely
*Report-Only mode is not optional. It is how you avoid breaking your organization.*

> 🟢 **Beginner** · Unit 4, Module 4.7

---

## What You Will Learn

- Why **Report-Only mode is critical** — it logs what a policy would do without actually enforcing it
- How to read **report-only results** in sign-in logs and understand what the policy would have done
- How to use **What-If tool** to predict policy impact before enabling
- How to measure **impact scope** — how many users would be affected by enforcing a policy
- When to **transition from Report-Only to Enforce** — the decision framework
- How to **enable a policy carefully** — steps to enforce safely without surprises

---

## Why This Matters

*SC-300: Implement authentication and access management — Test and validate Conditional Access policies safely*

CA policies can lock users out. A poorly tested MFA policy can block 30% of your organization. A device compliance policy enforced before users are enrolled can make critical apps inaccessible. This is not a lab anymore — this is production. One mistake breaks business.

Report-Only mode is how you test in production safely. The policy is active (logging what would happen), but not blocking anything. You collect data for days or weeks, analyze the impact, and only then enforce. Report-Only is not a "beta test" feature — it is a mandatory step in every serious CA rollout.

---

## 📊 Report-Only: The Three Signals

A CA policy in Report-Only generates three signals in sign-in logs:

**Signal 1: "Report-only — would allow"**
The policy condition was satisfied and would have allowed the sign-in. The user succeeded because the policy is not enforcing. If you enable Enforce, this sign-in would still be allowed.

**Signal 2: "Report-only — would require additional auth"**
The policy condition was not satisfied, but the policy would have required additional authentication (MFA, device enrollment, etc.). The user succeeded in the lab because the policy is not enforcing, but they did not complete the additional auth. If you enable Enforce, these users would be challenged.

**Signal 3: "Report-only — would block"**
The policy condition was not satisfied and the policy would have blocked the sign-in completely. The user succeeded in the lab because the policy is not enforcing, but if you enable Enforce, they would be blocked.

By analyzing these signals over 1-2 weeks, you answer critical questions:
- How many users would be blocked? (Count "would block" results)
- How many users need remediation? (Count "would require additional auth" results)
- Are there unexpected exclusions? (Check if some high-risk users are unexpectedly exempt)
- Are the logs showing the policy is working as intended? (Spot-check a few entries)

---

## 🎯 The Report-Only Decision Framework

| Users Affected | Action |
|---|---|
| 0-5% | Likely safe to enforce. This is expected — some users always need exceptions. |
| 5-20% | **Requires remediation plan.** Do not enforce until you have a plan to enroll/fix those users. Estimate 1-2 weeks for remediation. |
| 20-50% | **Major impact.** Slow rollout required. Start with a pilot group (e.g., IT department), measure, then expand weekly. |
| >50% | **Do not enforce.** The policy is blocking too many users. Re-evaluate the policy design or prerequisites (e.g., are devices enrolled yet?). |

Contoso's approach: Device compliance for SharePoint (Module 4.5) will likely affect 30-40% of users (those with unmanaged devices). Strategy: Enroll users in Intune first (Week 1-2), collect Report-Only data (Week 3), then enforce for compliant devices and create exclude groups for contractors who cannot enroll.

---

## 📋 How to Read Report-Only Results in Sign-In Logs

**Location:** Entra Admin Center → Protection → Conditional Access → **Sign-in logs**

Each sign-in entry shows a **Conditional Access** section with details:

```
Conditional Access:
├─ Policy: "Require phishing-resistant MFA for Azure Portal"
├─ Status: Report-only
├─ Result: Would Block
├─ Required: Authentication Strength = Phishing-resistant MFA
├─ Provided: Authenticator app push (does not meet requirement)
└─ Evaluation: User does not have phishing-resistant MFA configured
```

**Result statuses you will see:**

- ✅ **Satisfied** — policy condition passed, sign-in allowed
- ❌ **Would Block** — policy would have blocked, but Report-Only allowed it
- ⚠️ **Would require additional auth** — policy would have required additional challenge, but Report-Only allowed it
- 🚫 **Not Applied** — policy did not apply (user is excluded, wrong app, wrong location, etc.)

**Read these log entries to answer:**
1. Is the policy targeting the right users? (Check "Not Applied" — are some users unexpectedly exempt?)
2. Is the policy blocking the right sign-ins? (Check "Would Block" — are these expected?)
3. How many users would need to remediate? (Count "Would require additional auth" entries)

---

## 📱 What-If Tool: Predict Before Enforcing

The What-If tool lets you simulate a policy without waiting for real sign-ins to occur.

**Location:** Entra Admin Center → Protection → Conditional Access → **What If**

**What-If lets you ask:**
- "If I enforce this policy, what happens when Alex signs into Teams?"
- "What happens when a contractor signs into SharePoint?"
- "What if someone signs in from Germany?"

**What-If does NOT:**
- Execute a real sign-in (no audit log entry, no MFA challenge)
- Account for real-time device state (it evaluates based on current snapshot)
- Predict future changes (if a user enrolls in MFA tomorrow, What-If does not know)

**What-If is useful for:**
- Spot-checking a few key users/scenarios before enforcing
- Verifying policy logic is correct
- Training — showing stakeholders "here is what will happen"
- Excluding users proactively — "if this contractor signs into Azure Portal, what-if shows they would be blocked, so let's exclude them first"

---

## Lab 4.7.1 — Analyze Report-Only Data: Measure Impact of One Policy

**Tools:** Entra admin center
**Prerequisite:** Labs 4.3 and 4.4 complete (at least two CA policies created in Report-Only)

By now, you have created multiple CA policies in Report-Only mode:
- Lab 4.3: MFA for administrators
- Lab 4.4: Block sign-ins from disallowed countries
- Lab 4.5: Require compliant device for SharePoint
- Lab 4.6: Phishing-resistant MFA for Azure Portal, standard MFA for other apps

In this lab, you will analyze the impact data for one policy.

1. Open the Entra admin center. Navigate to **Protection** → **Conditional Access** → **Sign-in logs**.

2. You will see a list of sign-ins. By default, it shows the last 24 hours.

3. On the right side of the page, look for **Conditional Access** column. You should see policy names and results.

4. **Goal:** Find 3-5 sign-in entries for a specific policy (e.g., "Require MFA for administrators" or "Block sign-ins from disallowed countries"). For each entry, note:
   - **User:** Who signed in?
   - **App:** Which app were they accessing?
   - **Policy:** Which CA policy applied?
   - **Status:** Report-only
   - **Result:** Was it "Satisfied," "Would Block," or "Would require additional auth"?

5. Create a simple tally (mental or on paper):
   - **Satisfied:** 3 entries
   - **Would Block:** 0 entries
   - **Would require additional auth:** 1 entry

6. **Analyze:** If you were to enforce this policy, would 0 sign-ins be blocked? Would 1 user need to remediate (enroll in MFA)? This is the data that guides your enforcement decision.

**Example Report-Only Entry:**

```
Timestamp: 2024-03-01 10:30 AM
User: Alex Lee
App: Microsoft Teams
Policy: Require MFA for administrators
Status: Report-only
Result: Satisfied
Details: Alex is in the Administrators group. Alex signed in from a trusted location.
         MFA was required and provided. Policy condition passed.
         → If enforced: This sign-in would be allowed.
```

**Another Example:**

```
Timestamp: 2024-03-01 02:15 AM
User: Jordan Kim
App: Azure Portal
Policy: Require phishing-resistant MFA for Azure Portal
Status: Report-only
Result: Would Block
Details: Jordan attempted to sign into Azure Portal from Germany.
         Phishing-resistant MFA required. Jordan provided standard Authenticator app.
         → If enforced: This sign-in would be blocked. Jordan needs Windows Hello or FIDO2.
```

---

## Lab 4.7.2 — Use What-If Tool: Simulate Policy Impact for Key Users

**Tools:** Entra admin center
**Prerequisite:** Lab 4.7.1 complete

The What-If tool lets you test scenarios without waiting for them to occur naturally. You will simulate policy impact for key users/scenarios.

1. Open the Entra admin center. Navigate to **Protection** → **Conditional Access** → **What If**.

2. You will see a form with fields:
   - **User**
   - **App**
   - **Compliance state**
   - **Sign-in risk**
   - **Locations**

3. **Scenario 1: Administrator accessing Azure Portal (should pass phishing-resistant MFA requirement)**

   - User: Select your Global Administrator account (yourself)
   - App: **Azure Portal**
   - Leave other fields at default
   - Click **What If**

   **Expected result:** One of these:
   - ✅ All policies satisfied (if you have phishing-resistant MFA configured)
   - ⚠️ "Require phishing-resistant MFA for Azure Portal — Would Block" (if you don't have phishing-resistant MFA yet)

4. **Scenario 2: Non-admin user accessing Teams (should require standard MFA)**

   - User: Select **Alex Lee** (or another non-admin test user)
   - App: **Microsoft Teams**
   - Click **What If**

   **Expected result:**
   - ✅ "Require MFA for all other apps — Satisfied" (standard MFA required and available)

5. **Scenario 3: Contractor accessing Azure Portal (should be blocked)**

   - User: Select a contractor or guest account (if you have one)
   - App: **Azure Portal**
   - Click **What If**

   **Expected result:**
   - ❌ "Require phishing-resistant MFA for Azure Portal — Would Block"
   - This tells you: contractor cannot access Azure Portal unless they have phishing-resistant MFA. Decide: (a) exclude them, (b) enroll them in FIDO2, or (c) create a separate policy for contractors.

6. **Key observation:** What-If showed you the impact for specific scenarios without executing them. Use this to:
   - Spot-check your policies before enforcing
   - Decide which users need exclusions
   - Plan remediation (e.g., "we need to enroll contractors in FIDO2 before enforcing")

---

## Lab 4.7.3 — Transition One Policy from Report-Only to Enforce

**Tools:** Entra admin center
**Prerequisite:** Labs 4.7.1 and 4.7.2 complete

Now you will take one policy from Report-Only to Enforce. Choose the policy with the **lowest impact** (safest to enforce):

**Best candidates to enforce first:**
1. **MFA for administrators** (from Module 4.3) — usually affects only a small group, impact is well-understood
2. **Block sign-ins from disallowed countries** (from Module 4.4) — if you are in the US and most users are in the US, impact is minimal
3. **Device compliance for SharePoint** (from Module 4.5) — higher impact; wait if <70% of users are compliant
4. **Phishing-resistant MFA for Azure Portal** (from Module 4.6) — **too risky to enforce first** — you likely do not have phishing-resistant MFA deployed yet

**For this lab, enforce the MFA for administrators policy** (Module 4.3). This is the safest.

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. Find the policy **"Require MFA for all administrators"** (created in Module 4.3).

3. Click on it to open.

4. Scroll down to the **State** section.

5. Current state: **Report-only**

6. Change to: **On** (Enforce)

7. Click **Save**.

**Verification:**

8. The policy is now **Enforce** — any sign-in by an administrator not providing MFA will be blocked.

9. Navigate to **Sign-in logs** and look for administrator sign-ins in the last few minutes.

10. Find an administrator sign-in and click on it. Under **Conditional Access**, you should now see:
    - Policy: "Require MFA for all administrators"
    - Status: **Enforced** (not Report-only)
    - Result: **Satisfied** (MFA was provided, so sign-in allowed)

**Why this policy is safe to enforce first:**
- Administrators are tech-savvy and already using MFA (likely)
- The group is small (easier to support if issues arise)
- MFA is a well-understood, standard control (not a new technology like device compliance)
- If something breaks, it affects a small group (not all employees)

---

## Lab 4.7.4 — Test the Enforced Policy: Verify It Works Correctly

**Tools:** Browser, Entra admin center
**Prerequisite:** Lab 4.7.3 complete

Now that the MFA for administrators policy is **Enforce** (not Report-Only), verify it works as expected.

1. Sign out of the Entra admin center.

2. In a new browser tab, navigate to `entra.microsoft.com` and sign in as your Global Administrator.

3. **Observation:**
   - You will be challenged for MFA (same as before, but now it is enforced by policy)
   - Provide your MFA (push notification, code, biometric, etc.)
   - You will be signed in

4. Check the sign-in log:
   - Navigate back to **Protection** → **Conditional Access** → **Sign-in logs**
   - Find your recent sign-in as Global Administrator
   - Click on it
   - Under **Conditional Access**, the policy should show:
     - "Require MFA for all administrators"
     - Status: **Enforced**
     - Result: **Satisfied** (MFA was provided)

5. **If the result showed "Satisfied," the policy is working correctly:**
   - The policy checked that you are an administrator
   - The policy required MFA
   - You provided MFA
   - Access was allowed

6. **If you were blocked (not blocked in this lab, but this is what enforced policies do):**
   - You would see an error message
   - The log would show "Denied" instead of "Satisfied"

**Key takeaway:** An enforced CA policy that "Satisfied" is not visible to the user as friction — the policy approved access. Only policies that "Would Block" or "Would require additional auth" create friction (MFA challenge, device enrollment, etc.).

---

## Scenario Check

Contoso has one CA policy now in Enforce mode: **Require MFA for all administrators**. The Report-Only data showed zero impact — all administrators were already using MFA. This was the safest policy to enforce first.

The other policies remain in Report-Only:
- **Block sign-ins from disallowed countries** — 2-3 entries over 1 week showing "Would Block" (suspicious foreign sign-ins) → **Safe to enforce next**
- **Require compliant device for SharePoint** — 40% "Would Block" — needs more prep (user enrollment in Intune) → **Enforce in 2-3 weeks**
- **Phishing-resistant MFA for Azure Portal** — 85% "Would Block" — very few administrators have Windows Hello configured yet → **Enforce in 4+ weeks, after enrollment campaign**

Jordan Kim documents:
> "We have progressively enabled CA policies based on Report-Only impact data. The admin MFA policy is now enforced with zero impact. Over the next two weeks, we will collect more data on the geo-block and device compliance policies. Around week 3-4, we will start a Windows Hello enrollment campaign for administrators, then enable the Azure Portal phishing-resistant MFA policy."

---

## Check Your Understanding

- [ ] Explain the difference between "Report-only — would block" and "Enforced — denied" in the sign-in logs.
- [ ] You have a CA policy in Report-Only. It shows 15% of users would be blocked. What is your next step? Why not enforce immediately?
- [ ] Use the What-If tool: simulate yourself signing into Azure Portal with your current authentication methods. What result do you see? Why?
- [ ] You enforced a device compliance policy and now 5% of users cannot access SharePoint. Was this a mistake? What should have happened first?
- [ ] In the sign-in logs, you see 50 administrator sign-ins. 48 show "Satisfied" for MFA policy, 2 show "Would require additional auth." What does this mean?
- [ ] Describe the safest approach to enforcing a CA policy: which policy should you enforce first, and why?

---

## Common Mistakes

**Enforcing a policy without running Report-Only first.**
A policy enforced without 1-2 weeks of Report-Only data is a risk. You do not know how many users you are blocking, what apps are affected, or if there are edge cases (contractors, partner accounts, legacy systems). Always run Report-Only first. Period.

**Misreading "Would block" as "will block" in Report-Only.**
In Report-Only mode, "Would block" means the policy would block IF enforced, but it is not enforcing, so the user succeeded. Do not confuse "Would block" with "Denied." Report-Only does not block anything.

**Enforcing multiple policies at once.**
If you enforce 3 policies on Monday and something breaks on Tuesday, you have 3 suspects. Enforce one policy at a time, wait 2-3 days, verify, then enforce the next. This isolates problems.

**Not excluding contractor and partner accounts from strict policies.**
Contractors and partners often cannot enroll in Intune or have phishing-resistant MFA. If you enforce device compliance or phishing-resistant MFA without excluding them, they are locked out. Decide early: (a) which policies apply to external identities, or (b) create a separate policy with lighter controls for them.

**Forgetting the break-glass account needs exclusion.**
The emergency access account must be excluded from all CA policies. If it is subject to a policy requiring phishing-resistant MFA and does not have it configured, you cannot use it to regain control if something breaks. Verify your CA-Exclusion-BreakGlass exclusion is on every policy.

**Using What-If results as a substitute for Report-Only data.**
What-If is a spot-check tool — useful for specific scenarios, not for understanding patterns. Real Report-Only data over 1-2 weeks shows you actual user behavior, business hours patterns, unexpected locations, contractor activity, etc. Use What-If to validate your thinking, but rely on Report-Only logs for enforcement decisions.

---

## What's Next

Module 4.8 is the **production CA policy stack for Contoso** — a capstone lab where you will design and implement a realistic multi-policy strategy: MFA for admins (Enforce), device compliance for SharePoint (Enforce), phishing-resistant MFA for Azure Portal (Report-Only pending enrollment), and geo-block policies. You will also learn how to monitor policies over time, adjust exclusions, and handle exceptions.

By the end of Unit 4, Contoso's CA strategy will be a working, production-grade system — tested, monitored, and safe.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.6 — App Conditions](../4-6-app-conditions/4-6-app-conditions.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.8 — Production CA Policy Stack →](../4-8-production-ca-policy-stack/4-8-production-ca-policy-stack.md)
