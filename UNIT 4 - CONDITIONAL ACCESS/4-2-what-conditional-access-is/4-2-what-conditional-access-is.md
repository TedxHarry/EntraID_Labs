# What Conditional Access Is — The IF-THEN Engine
*Every sign-in gets evaluated against every applicable policy. The result is the most restrictive outcome of all of them.*

> 🟢 **Beginner** · Unit 4, Module 4.2

---

## What You Will Learn

- Explain the IF-THEN model that every Conditional Access policy follows
- Describe how multiple policies interact on the same sign-in
- Identify the five assignment conditions and three grant controls
- Use the What-If tool to simulate a policy outcome before the policy exists
- Find and read the Conditional Access result in a sign-in log

---

## Why This Matters

*SC-300: Implement authentication and access management — Plan and implement Conditional Access policies*

Conditional Access is the most tested area in the SC-300 exam and the most critical skill in real-world Entra ID administration. Every policy you build for the rest of this series — MFA, device compliance, location-based access, application-specific controls — is a Conditional Access policy. Understanding the model before building policies prevents the most common mistake: creating policies that interact in unexpected ways and either block legitimate users or leave gaps attackers can walk through.

---

## 🔀 The IF-THEN Model

Every Conditional Access policy has two parts:

**IF (Assignments) — the conditions that trigger the policy:**
- **Users and groups** — who the policy applies to (all users, a specific group, an admin role)
- **Target resources** — which apps or services (all cloud apps, specific apps like SharePoint)
- **Conditions** — additional signals: device state, sign-in risk level, location, client app type, user risk level

**THEN (Access controls) — what happens when the conditions are met:**
- **Grant** — allow access, but require something (MFA, compliant device, specific authentication strength)
- **Block** — deny access entirely
- **Session** — apply a session control (sign-in frequency, app enforced restrictions)

A policy only applies to a sign-in when ALL the IF conditions are met. If the user is not in the target group, the policy is skipped. If the app is not in the target resources, the policy is skipped. Only when every condition matches does the THEN execute.

---

## 🔢 Multiple Policies on One Sign-In

When Alex Lee signs in to SharePoint from an unknown location, Entra ID evaluates every enabled Conditional Access policy against that sign-in — not just one. If three policies apply, all three grant controls are applied simultaneously.

**Example:**
- Policy A: All users → All apps → Require MFA
- Policy B: Finance group → SharePoint → Require compliant device
- Policy C: All users → All apps → Block if sign-in risk is High

If Alex is in the Finance group, all three policies apply. Alex must satisfy MFA (Policy A), have a compliant device (Policy B), and not have a High sign-in risk (Policy C). If any one of these conditions is not met, access is denied.

The result is always the most restrictive combination of all applicable policies.

> 🔑 **Key concept:** There is no "allow" policy that overrides a "block" policy. If any applicable policy results in Block, the sign-in is blocked. The only way to allow access for a specific user despite a blocking policy is to exclude that user from the blocking policy — not to create an Allow policy.

---

## 📋 Policy Assignments — The Five Conditions

| Condition | What it evaluates | Common use |
|---|---|---|
| **Users and groups** | Is this user (or their group/role) in scope? | Scope to all users, admins only, or specific departments |
| **Target resources** | Is this app or service in scope? | All apps, or specific apps like SharePoint/Exchange/Azure portal |
| **Network / Locations** | Is the sign-in from a named location? | Trusted networks vs unknown, specific countries |
| **Device state** | Is the device compliant? Domain-joined? Registered? | Managed vs unmanaged device conditions |
| **Client apps** | Is the client using modern auth or legacy auth? | Block legacy protocols (SMTP Auth, IMAP, POP3) |
| **Sign-in risk** (P2) | Is the sign-in flagged as risky by Identity Protection? | Block or challenge high-risk sign-ins |
| **User risk** (P2) | Is the user's account flagged as compromised? | Require password change for high-risk users |

---

## 🎯 Grant Controls — What You Can Require

| Control | What it means |
|---|---|
| **Require multifactor authentication** | User must complete a second factor |
| **Require authentication strength** | User must use a specific method strength (phishing-resistant, MFA, password + something) |
| **Require compliant device** | Device must be managed by Intune and marked compliant |
| **Require hybrid Azure AD joined device** | Device must be joined to on-premises AD and registered with Entra ID |
| **Require approved client app** | Must use Microsoft-managed app (Outlook, Teams) not a generic browser |
| **Require app protection policy** | MAM policy must be applied to the app (Intune mobile app management) |
| **Require password change** | User must change their password before proceeding |
| **Block access** | No access, no alternative — the sign-in is rejected |

Multiple grant controls can be combined with AND (require all) or OR (require any one).

---

## Lab 4.2.1 — Explore the Conditional Access Policy Blade

**Tools:** Entra admin center
**Prerequisite:** Security Defaults disabled (Module 4.1)

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Protection** → **Conditional Access**.

3. The Conditional Access overview page shows your current policies. In a fresh tenant after disabling Security Defaults, the policy list may be empty or contain only the report-only SharePoint sign-in frequency policy from Module 3.6.

4. Click on the **What If** tool in the left menu (or in the top toolbar of the Conditional Access blade).

5. The What If tool opens. This is where you simulate what would happen to a sign-in without actually performing it.

---

## Lab 4.2.2 — Use the What-If Tool to Simulate Sign-In Scenarios

**Tools:** Entra admin center → Conditional Access → What If
**Prerequisite:** Lab 4.2.1 complete

Run five simulations. For each one, set the inputs and record the result.

**Simulation 1: Alex signs in to All cloud apps from anywhere**

1. In the What If tool, set:
   - **User:** Alex Lee
   - **Cloud apps, actions, or authentication context:** Select **All cloud apps**
   - Leave all other fields at default

2. Click **What If**. Review the results.

3. If no policies are in Enforce mode, the result shows: no policies applied — access would be granted without any controls. This is the current state — no protection.

**Simulation 2: What-If with the Sign-In Frequency policy (Report-only)**

4. Change **User** to Alex Lee, **Cloud apps** to **Office 365 SharePoint Online**.

5. Click **What If**. The Finance SharePoint re-auth policy from Module 3.6 appears in the results. Its state shows **Report-only** — it would have applied in Enforce mode, but currently doesn't enforce.

**Simulation 3: Simulate a sign-in from a high-risk location**

6. Change inputs:
   - **User:** Alex Lee
   - **Cloud apps:** All cloud apps
   - **Sign-in risk:** High

7. Click **What If**. Without a risk-based CA policy, no policies apply to high sign-in risk. The result: access would be granted despite High risk. This is another gap — closed in Module 4.8.

**Simulation 4: Simulate Jordan Kim (IT Administrator) signing in to the Azure portal**

8. Change:
   - **User:** Jordan Kim
   - **Cloud apps:** Select **Microsoft Azure Management** (or search for it)

9. Click **What If**. No policies apply — Jordan can access the Azure portal without MFA or any control. This gap is closed in Module 4.3.

**Simulation 5: Record your findings**

10. In a text block at the bottom of this simulation session (or a separate notes file), write down: for each simulation, what the current result is and what the expected result should be after the CA stack is complete in Module 4.8.

---

## Scenario Check

The What-If tool revealed Contoso's current state after disabling Security Defaults: no active Conditional Access policies. Alex Lee can sign in to any app from any location with only a password. Jordan Kim can access the Azure portal without MFA. High-risk sign-ins are not blocked.

The simulations also showed the one report-only policy that exists: the SharePoint sign-in frequency from Module 3.6. The What-If correctly identified it as applicable but not enforced.

These results are the baseline that the next six modules will fix. By Module 4.8, every simulation that currently shows "no policies applied" will show specific policies enforcing specific controls.

---

## Check Your Understanding

- [ ] Open the What-If tool and run a simulation for Morgan Chen signing in to SharePoint from a trusted location — state the current result
- [ ] From memory: state the rule about multiple policies applying to one sign-in — what is the result of Policy A requiring MFA and Policy B blocking?
- [ ] Name three Grant controls available in Conditional Access
- [ ] Explain why there is no "Allow" policy — how do you allow a specific user past a blocking policy?
- [ ] State the difference between a Grant control that requires MFA and one that requires Authentication Strength
- [ ] From memory: name the five conditions in the Assignments section of a CA policy

---

## Common Mistakes

**Creating a "catch-all Allow" policy to override a Block policy.**
This is not how CA works. An Allow policy does not exist — Grant controls require something, they don't unconditionally allow. If a Block policy applies to a user and you want that user to have access, exclude them from the Block policy.

**Assuming policies run in priority order.**
There is no priority order. All applicable policies run. All their grant controls must be satisfied simultaneously. The order of policies in the list has no effect on evaluation.

**Targeting All users in an early policy without excluding the break-glass account.**
If you create a policy targeting All users that blocks or requires something the break-glass account cannot satisfy, you may lose emergency access. Always add `CA-Exclusion-BreakGlass` as an excluded group before switching any policy to Enforce.

**Not using the What-If tool before enabling a policy.**
The What-If tool is free, fast, and catches unexpected interactions. Never switch a policy from Report-only to Enforce without running a What-If simulation for your key scenarios first.

---

## What's Next

Module 4.3 builds the first real CA policy: require MFA for all administrator roles. This is the most urgent gap opened by disabling Security Defaults — your admin accounts can currently sign in without MFA. This module walks through building, testing in Report-Only, and then enforcing the policy.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.1 — Security Defaults](../4-1-security-defaults/4-1-security-defaults.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.3 — Your First CA Policy →](../4-3-your-first-ca-policy/4-3-your-first-ca-policy.md)
