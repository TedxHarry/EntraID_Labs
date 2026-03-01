# Scenario Lab: Secure Your Tenant's Authentication — End to End
*No step-by-step. You have the tools. Find the gaps. Fix them.*

> 🟡 **Intermediate** · Unit 3, Module 3.7 · ⏱️ 90 minutes

---

## What This Lab Is

This is a Scenario Lab. You receive a brief describing Contoso's current authentication state. You assess the gaps, produce a before-state report, fix everything, and produce an after-state report.

There are no numbered steps. You apply what you learned in Modules 3.1 through 3.6. If you need to look something up, use the earlier modules — not a search engine. The goal is to practice doing the work, not following instructions.

---

## The Brief

Contoso's new CISO has completed an initial review of the Microsoft 365 environment. The findings are not good.

**Finding 1 — SMS is enabled for all users.**
The authentication method policy currently allows any user to register SMS as an MFA method. Three users have SMS as their only registered method. SMS is a phishing and SIM-swap risk and does not meet the CISO's acceptable authentication standard.

**Finding 2 — No one except Alex Lee has MFA registered.**
SSPR is enabled, but only Alex Lee has MFA registered through the Authenticator app. If a CA policy requiring MFA is enforced today, four out of five users would be blocked at sign-in.

**Finding 3 — There is no sign-in frequency control on the Finance SharePoint site.**
Finance users can access the Finance SharePoint document library indefinitely without re-authenticating. Given the sensitivity of financial data, the CISO requires re-authentication every 4 hours for Finance users accessing SharePoint.

**Finding 4 — Session revocation behavior is not understood by the IT team.**
Jordan Kim, the IT Administrator, does not know how to revoke a user's active sessions or how long access persists after revocation. The CISO wants a written procedure in place.

---

## Your Objectives

Fix all four findings. Produce a brief written summary (as a comment at the end of this file, or in a separate `3-7-notes.md` file) that records:

1. What was wrong before (the before state)
2. What you changed (the actions taken)
3. What the after state looks like (verification)

---

## Lab 3.7.1 — Audit and Remediate SMS Access

**Time:** ~15 minutes
**Tools:** Entra admin center, PowerShell

Complete these tasks without step-by-step guidance:

- Navigate to the Authentication Methods Policy and find the SMS entry. Document its current configuration (who can use it).
- Navigate to the user registration details report. Find all users who have SMS as a registered method. Record their names.
- Restrict SMS to the `BreakGlass-SMS` group only (you configured this in Module 3.2 — confirm it is still correctly set).
- Confirm that regular Contoso users cannot register SMS as a new method.

**Expected outcome:** SMS is enabled only for the BreakGlass-SMS group. Alex Lee, Morgan Chen, Jordan Kim, River Patel, and Sam Taylor cannot register or use SMS.

---

## Lab 3.7.2 — Get All Users MFA-Ready

**Time:** ~25 minutes
**Tools:** Entra admin center, PowerShell

Complete these tasks:

- Use PowerShell to pull the MFA registration status for all five Contoso users. Identify who has zero MFA methods registered.
- Issue Temporary Access Passes to each user who has no MFA registered (Morgan Chen, Jordan Kim, River Patel). Set each TAP to 120 minutes, one-time use.
- Using separate InPrivate browser windows (one per user), sign in as each unregistered user using their TAP. Navigate to `aka.ms/mysecurityinfo` and register Microsoft Authenticator for each user.
- After all registrations are complete, run the PowerShell MFA status check again. All five Contoso users should now show at least one registered MFA method.

**Expected outcome:** All five Contoso users show MFA registered: Yes in the User registration details report.

---

## Lab 3.7.3 — Implement Sign-In Frequency for Finance SharePoint

**Time:** ~20 minutes
**Tools:** Entra admin center

Complete these tasks:

- Create a Conditional Access policy targeting Finance users (use the Finance-Dynamic or Finance-All group) and scoped to SharePoint Online.
- Set the Sign-In Frequency to 4 hours.
- Start the policy in Report-Only mode.
- Sign in as Alex Lee (Finance department) and navigate to SharePoint Online.
- Find the sign-in log entry and verify that the CA policy appears in the Conditional Access tab with a Report-only result.
- Switch the policy to **Enabled** (Enforce).

**Expected outcome:** The policy `Finance — SharePoint re-auth 4 hours` is in Enforce mode, targeting Finance-Dynamic (or Finance-All) and SharePoint Online. The sign-in log shows the policy applied for Alex Lee.

---

## Lab 3.7.4 — Create the Session Revocation Procedure

**Time:** ~10 minutes
**Tools:** Entra admin center, this file or a notes document

Write a procedure that Jordan Kim (IT Administrator) can follow to revoke a user's sessions when a security incident is suspected. The procedure must include:

1. Where to navigate in the Entra admin center to revoke sessions for a specific user
2. What happens immediately after revocation (what changes, what doesn't)
3. How long access persists after revocation without CAE
4. How to verify whether CAE is active for the resources the user was using
5. What additional action to take if CAE is not active and immediate access termination is required

Write the procedure here (below the heading):

---

**[Write your procedure here — no guidance provided]**

---

## Verification Checklist

After completing all four labs, verify each item:

- [ ] **SMS:** Navigate to Authentication Methods Policy → SMS. Confirm target is `BreakGlass-SMS` only, not All users.
- [ ] **MFA Coverage:** Navigate to Protection → Authentication methods → User registration details. Filter by MFA registered: No. Confirm zero results for the five Contoso users.
- [ ] **Sign-In Frequency Policy:** Navigate to Protection → Conditional Access → Policies. Find the Finance SharePoint policy. Confirm it is in Enabled (Enforce) state.
- [ ] **Audit log:** Navigate to Identity → Monitoring & health → Audit logs. Find entries for TAP issuance for Morgan, Jordan, and River. Confirm the date/time and initiated-by (should be your admin account).
- [ ] **Sign-in logs:** Find a recent Alex Lee sign-in to SharePoint Online. Open the Conditional Access tab. Confirm the Finance SharePoint re-auth policy shows as Applied (not Report-only).
- [ ] **Session revocation procedure:** Confirm the procedure is written and covers all five required points.

---

## Scenario Check

Contoso's authentication security posture changed significantly during this lab. The before state: SMS available to all users, only one user (Alex) with MFA registered, no session controls on Finance data, and no documented revocation procedure.

After:
- SMS is restricted to the break-glass group only. No standard user can register or use SMS.
- All five Contoso users have Microsoft Authenticator registered. If a CA policy requiring MFA is enforced today, all five pass.
- Finance users accessing SharePoint are required to re-authenticate every 4 hours. Alex Lee's next SharePoint session after the 4-hour window will trigger a fresh sign-in prompt.
- Jordan Kim now has a documented procedure for session revocation — covering both immediate action and the CAE-dependent limitation.

The CISO's four findings are resolved.

---

## What's Next

Unit 4 — Conditional Access. This is the largest and most critical unit in the series. Everything you built in Unit 3 (MFA, authentication methods, session controls) becomes a Conditional Access policy in Unit 4. The work you did in this module sets up the policy stack for Unit 4 — specifically, all five Contoso users now have MFA registered, which means a CA policy requiring MFA will not block everyone on day one.

Start with Module 4.1 — Security Defaults: Understanding What You're Replacing.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 3.6 — Token Lifetime and Session Management](../3-6-token-lifetime-session-management-cae/3-6-token-lifetime-session-management-cae.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.1 — Security Defaults →](../../UNIT 4 - CONDITIONAL ACCESS/4-1-security-defaults/4-1-security-defaults.md)
