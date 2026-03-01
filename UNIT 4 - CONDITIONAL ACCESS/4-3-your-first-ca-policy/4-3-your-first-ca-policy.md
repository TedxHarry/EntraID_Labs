# Your First CA Policy — Require MFA for Administrators
*Admin accounts are the highest-value target. Protect them first.*

> 🟢 **Beginner** · Unit 4, Module 4.3 · ⏱️ 45 minutes

---

## What You Will Learn

- Build a Conditional Access policy from scratch with correct assignments and grant controls
- Understand why Report-Only mode is mandatory before enforcement
- Read the CA result from a sign-in log and understand what report-only outcome means
- Switch the policy from Report-Only to Enforce
- Sign in as a Global Administrator and experience the MFA prompt the policy creates

---

## Why This Matters

*SC-300: Implement authentication and access management — Plan and implement Conditional Access policies*

Global Administrator is the most powerful role in your tenant. An attacker who compromises a Global Administrator can disable every policy you build, create backdoor accounts, and export your entire user directory. MFA on admin accounts does not prevent all attacks — but it stops the most common attack path: credential phishing. An attacker who phishes an admin's password still cannot sign in without the second factor. This is the first policy you build because admin accounts are the first things attackers target after Security Defaults are off.

---

## 🏗️ Policy Structure — What You Will Build

**Policy name:** `Require MFA — All Administrators`

| Component | Configuration |
|---|---|
| **Users — Include** | Select users and groups → Directory roles → Select 8 admin roles |
| **Users — Exclude** | Groups → CA-Exclusion-BreakGlass |
| **Target resources** | All cloud apps |
| **Conditions** | None (applies to all locations, all devices, all client apps) |
| **Grant** | Require multifactor authentication |
| **Session** | None |
| **State** | Report-only first, then Enforce |

**Admin roles to include (the eight most privileged):**
1. Global Administrator
2. Privileged Role Administrator
3. Conditional Access Administrator
4. Security Administrator
5. Exchange Administrator
6. SharePoint Administrator
7. Authentication Administrator
8. User Administrator

---

## Lab 4.3.1 — Build the Policy in Report-Only Mode

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Security Defaults disabled (Module 4.1). CA-Exclusion-BreakGlass group exists.

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Protection** → **Conditional Access** → **Policies**.

3. Click **+ New policy**.

4. Name the policy: `Require MFA — All Administrators`

**Set Users — Include:**

5. Under **Users**, click **0 users and groups selected**.

6. Select the **Include** tab. Select **Select users and groups** (not All users — you are targeting roles, not everyone).

7. Check the box for **Directory roles**.

8. In the roles search, find and select these roles:
   - **Global Administrator**
   - **Privileged Role Administrator**
   - **Conditional Access Administrator**
   - **Security Administrator**
   - **Exchange Administrator**
   - **SharePoint Administrator**
   - **Authentication Administrator**
   - **User Administrator**

9. Click **Select**.

**Set Users — Exclude:**

10. Click the **Exclude** tab on the Users section.

11. Check **Users and groups**. Click **Select excluded users and groups**.

12. Search for and select **CA-Exclusion-BreakGlass**. Click **Select**.

**Set Target Resources:**

13. Under **Target resources**, click **No target resources selected**.

14. Select **Cloud apps**. Select **All cloud apps**. Click **Done**.

**Set Grant:**

15. Under **Access controls** → **Grant**, click **0 controls selected**.

16. Select **Require multifactor authentication**. Click **Select**.

**Set State:**

17. At the bottom of the policy, under **Enable policy**, select **Report-only**.

18. Click **Create**.

The policy is saved. You should see it in the policy list with a status of **Report-only**.

---

## Lab 4.3.2 — Test in Report-Only and Read the Sign-In Log

**Time:** 15 minutes
**Tools:** Entra admin center, InPrivate browser
**Prerequisite:** Lab 4.3.1 complete

1. Open an InPrivate browser. Navigate to `entra.microsoft.com`.

2. Sign in with your Global Administrator account. Sign in using only the password — do not complete MFA if prompted separately.

3. After sign-in, return to your main admin browser.

4. Navigate to **Identity** → **Monitoring & health** → **Sign-in logs**.

5. Filter by your Global Administrator UPN. Find the most recent sign-in entry. Click on it.

6. Select the **Conditional Access** tab. Find the policy `Require MFA — All Administrators`.

7. The result column shows: **Report-only: MFA required** (or **Report-only: Success** meaning the policy would have applied and required MFA, but did not enforce it in this mode).

8. The detail pane shows what the policy evaluation found:
   - **Policy applied:** Yes
   - **Result:** MFA required (report-only)

This confirms the policy is evaluating correctly and would have required MFA if it were in Enforce mode.

---

## Lab 4.3.3 — Switch to Enforce and Experience MFA

**Time:** 10 minutes
**Tools:** Entra admin center, InPrivate browser, phone with Authenticator
**Prerequisite:** Lab 4.3.2 complete. Microsoft Authenticator is registered for your admin account.

1. Navigate to **Protection** → **Conditional Access** → **Policies** → click on **Require MFA — All Administrators**.

2. Scroll to **Enable policy** at the bottom. Change from **Report-only** to **On**.

3. Click **Save**.

> ⚠️ **Warning:** This policy is now active. Your Global Administrator account must complete MFA on every sign-in. Ensure your Authenticator app is available. If you cannot complete MFA, use the break-glass account to sign in and revert the policy to Report-only.

4. Open an InPrivate browser. Navigate to `entra.microsoft.com`.

5. Sign in with your Global Administrator account and password.

6. After entering the password, the MFA prompt appears: "Open Microsoft Authenticator on your phone."

7. Complete MFA using the Authenticator app. Access is granted.

8. Return to the sign-in log in your main browser. Find the most recent sign-in. On the **Conditional Access** tab, the policy now shows **Success** (instead of Report-only) — the MFA requirement was enforced and satisfied.

---

## Scenario Check

Jordan Kim now requires MFA to sign in to any cloud app with the IT Administrator role. Your Global Administrator account requires MFA. The break-glass account is excluded — confirmed by the policy's exclusion group.

Alex Lee, Morgan Chen, and River Patel are not in any of the eight listed admin roles, so this policy does not apply to them. Their sign-ins are still unprotected until Module 4.8 builds the All Users MFA policy.

The sign-in log shows two entries for the Global Administrator account: the report-only test (result: would have required MFA) and the enforced sign-in (result: MFA required, satisfied). Both entries are visible in the Conditional Access tab — confirming the policy lifecycle from Report-only to Enforce.

---

## Check Your Understanding

- [ ] Navigate to **Conditional Access** → **Policies** — confirm `Require MFA — All Administrators` shows status **On** (Enabled)
- [ ] Sign in as your Global Administrator in an InPrivate browser — confirm the MFA prompt appears after password entry
- [ ] Find the enforced sign-in in the sign-in log — on the Conditional Access tab, confirm the policy result shows **Success** (not Report-only)
- [ ] From memory: name the eight admin roles targeted in this policy
- [ ] Explain why the CA-Exclusion-BreakGlass group must be in the Exclude section — what would happen if it wasn't
- [ ] State what "Report-only: MFA required" means in a sign-in log result (vs "Success")

---

## Common Mistakes

**Targeting All users in the admin MFA policy instead of Directory roles.**
If you target All users and require MFA but some users have no MFA method registered, they are immediately locked out when the policy is enforced. Targeting directory roles limits the scope to admin accounts, who typically have MFA registered (or who you can remediate before enforcement).

**Switching to Enforce before verifying your admin MFA is registered.**
If your admin account has no MFA method and you enforce MFA, you lock yourself out of your own admin account. Test in Report-only first. Verify MFA registration. Then enforce.

**Forgetting to exclude the break-glass account.**
If the break-glass account is in a role targeted by the policy and not in the exclusion group, it is also required to complete MFA. The break-glass account exists for when MFA is not possible — excluding it from MFA requirements is the point.

**Adding roles one-by-one over time without a complete list.**
The eight roles listed in this module cover the most critical attack surface. In a real organization, review the complete list of privileged roles in Entra ID and include all of them in this policy. Omitting a role because "we don't use it much" creates a blind spot.

---

## What's Next

Module 4.4 covers Named Locations — a prerequisite for location-based CA conditions. You will define your trusted network (office IP, VPN range) and use it to require stricter controls for sign-ins from untrusted locations. The policy pattern: Admin from trusted network → MFA required. Admin from unknown location → Block or require stronger authentication.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.2 — What Conditional Access Is](../4-2-what-conditional-access-is/4-2-what-conditional-access-is.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.4 — Named Locations →](../4-4-named-locations/4-4-named-locations.md)
