# Security Defaults — Understanding What You're Replacing
*Before you build Conditional Access policies, you must disable the baseline Microsoft manages. Here is exactly what you are removing — and what you must rebuild.*

> 🟢 **Beginner** · Unit 4, Module 4.1

---

## What You Will Learn

- Explain what Security Defaults are and what they enforce on a tenant
- State why Security Defaults and Conditional Access policies cannot coexist
- Create an emergency access account before disabling Security Defaults
- Disable Security Defaults safely
- Document exactly what you removed and what must be rebuilt with CA policies

---

## Why This Matters

*SC-300: Implement authentication and access management — Implement Conditional Access policies*

Every new Entra ID tenant has Security Defaults enabled. Security Defaults provide a baseline of identity security controls without requiring any configuration. They are good enough for a small organization that cannot manage Conditional Access policies. They are not configurable — you cannot exclude groups, add conditions, or fine-tune anything. When you need to build a real Conditional Access policy stack, you must disable Security Defaults first. Disabling them removes the baseline. You are responsible for rebuilding everything they were providing with your own policies.

---

## 🛡️ What Security Defaults Enforce

When Security Defaults are enabled, three things happen automatically across your tenant with no additional configuration:

| What it does | Who it applies to | What the user experiences |
|---|---|---|
| **Require MFA registration** | All users | Within 14 days of first sign-in, all users are prompted to register an authentication method |
| **Require MFA for admin roles** | Global Administrator, Privileged Role Administrator, Conditional Access Administrator, Exchange Administrator, SharePoint Administrator, Security Administrator | Every sign-in to any app requires MFA — no exceptions |
| **Block legacy authentication** | All users | Sign-in attempts using SMTP Auth, POP3, IMAP, Exchange ActiveSync without modern authentication are blocked |

That is it. Security Defaults are intentionally simple. They cannot be customized, scoped by group, or made conditional.

---

## ⚠️ Why You Cannot Mix Security Defaults and Conditional Access

Microsoft enforces a hard rule: if Security Defaults are enabled, you cannot create and enforce Conditional Access policies. You can create them in Report-Only mode, but the Enforce option is unavailable. The reason: CA policies and Security Defaults overlap on MFA enforcement. Running both creates unpredictable results — users get double MFA prompts, exclusion groups are ignored, and troubleshooting becomes impossible.

The moment you disable Security Defaults, CA policies can be enforced. The moment you re-enable Security Defaults, all enforced CA policies are suspended.

> ⚠️ **Warning:** When you disable Security Defaults, your tenant immediately loses the three protections listed above. Legacy authentication attacks become possible. Non-admin users can sign in without MFA. This is a temporary gap — you must close it by enforcing CA policies in the same session or immediately after. Never disable Security Defaults and leave the gap open overnight.

---

## 🚨 Emergency Access Account — Create Before Doing Anything Else

An emergency access account is a cloud-only Global Administrator account used only when normal sign-in is impossible — when all CA policies are misconfigured, when MFA is broken for all admins, when your normal admin account is locked.

Before disabling Security Defaults (or before enforcing any CA policy), you must have an emergency access account that is excluded from all policies. Without it, a misconfigured CA policy can lock every admin out of the tenant.

Requirements for an emergency access account:
- Cloud-only — not synced from on-premises AD
- UPN format: use a `.onmicrosoft.com` domain address (not your custom domain — custom domain DNS could fail)
- Authentication: use a FIDO2 hardware key or a very long (25+ character) random password stored in a secure vault — not the Authenticator app (phone can break)
- Never used for day-to-day tasks
- Sign-in alerts set up (if alert fires, investigate immediately)
- Excluded from every CA policy that uses Block or requires specific authentication methods

---

## Lab 4.1.1 — Check Whether Security Defaults Are Currently Enabled

**Tools:** Entra admin center

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Identity** → **Overview** → **Properties** (select the **Properties** tab on the Overview page, not the gear icon).

3. Scroll to the bottom of the Properties page. Find the section **Manage security defaults**.

4. Click **Manage security defaults**. A panel opens on the right showing the Security defaults setting.

5. Note the current state: **Enabled** or **Disabled**. In a new developer tenant, it is usually **Enabled**.

6. Close the panel without making changes. Proceed to Lab 4.1.2.

---

## Lab 4.1.2 — Create an Emergency Access Account

**Tools:** Entra admin center
**Prerequisite:** Lab 4.1.1 complete

1. Navigate to **Identity** → **Users** → **All users** → **New user** → **Create new user**.

2. Fill in the fields:
   - **User principal name:** `breakglass@yourdomain.onmicrosoft.com` (use your `.onmicrosoft.com` domain — not a custom domain)
   - **Display name:** `Break Glass Admin`
   - **Password:** Click **Let me create the password**. Generate a 25+ character random password (use a password manager or combine five random words). Record this password in a secure location — a password manager or offline secure vault. Never store it in cloud services.

3. Click **Create**.

4. Open the newly created account. Navigate to **Assigned roles** → **+ Add assignments**.

5. Search for **Global Administrator**. Select it. Click **Add**. The break-glass account now has Global Administrator.

**Create an exclusion group:**

6. Navigate to **Identity** → **Groups** → **All groups** → **New group**.

7. Create:
   - **Group type:** Security
   - **Group name:** `CA-Exclusion-BreakGlass`
   - **Group description:** `Emergency access accounts excluded from all CA block policies`
   - **Membership type:** Assigned
   - **Members:** Add **Break Glass Admin**

8. Click **Create**.

9. Add your primary admin account to this group as well — so both accounts are excluded from block policies.

> 💡 **Tip:** Add the `CA-Exclusion-BreakGlass` group as an excluded group in every CA policy you create that uses Block or requires specific authentication methods. This prevents lock-out. Every CA policy in the next seven modules will include this exclusion.

---

## Lab 4.1.3 — Disable Security Defaults

**Tools:** Entra admin center
**Prerequisite:** Lab 4.1.2 complete — emergency access account and exclusion group exist

1. Navigate to **Identity** → **Overview** → **Properties** → **Manage security defaults**.

2. The Security defaults panel opens. Set the toggle to **Disabled**.

3. You will be prompted to select a reason for disabling. Select **My organization is using Conditional Access**. This is the correct reason — you are disabling to build a proper CA policy stack.

4. Click **Save**.

5. The confirmation notification appears: "Security defaults have been disabled."

**Document what you removed:**

6. In a new text file or comment block, record these three items that Security Defaults was providing that you have now removed:
   - **Legacy authentication blocking** — SMTP Auth, POP3, IMAP, EAS sign-ins are now possible again. You must block them in Module 4.8 with a CA policy.
   - **MFA for admin roles** — Your Global Administrator can now sign in without MFA. You must require MFA for admin roles in Module 4.3.
   - **MFA registration prompt** — New users are no longer automatically prompted to register MFA within 14 days. You must handle this through CA policies or registration campaigns.

---

## Lab 4.1.4 — Verify Security Defaults Are Off and Review the Gap

**Tools:** Entra admin center

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. Click **+ New policy**. Note that the **State** toggle at the bottom now shows three options: **Report-only**, **On**, and **Off**. When Security Defaults were enabled, the **On** option was greyed out. It is now available.

3. Click **Discard** — do not save this test policy.

4. Navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Sign-in logs** (or via Monitoring).

5. Attempt to sign in as Alex Lee in an InPrivate browser — sign in with only a password, no MFA prompt expected (because Security Defaults MFA requirement is now removed).

6. Alex can sign in with a password alone. This is the gap. It will be closed in Module 4.3 when you enforce an MFA CA policy.

---

## Scenario Check

Contoso's Security Defaults have been disabled. The emergency access account (`breakglass@yourdomain.onmicrosoft.com`) exists with Global Administrator access, is excluded from all future CA block policies via the `CA-Exclusion-BreakGlass` group, and its credentials are stored securely offline.

The three baseline protections that Security Defaults provided are now gone:
- Legacy authentication attacks against Contoso are now possible
- Admin accounts (including Jordan Kim, when Jordan receives the IT Administrator role) can sign in without MFA
- New users receive no MFA registration prompt

These gaps will be closed in Modules 4.3, 4.6, and 4.8. Alex Lee signing in with a password-only was the visible evidence of the gap. By the end of Module 4.8, that sign-in will be blocked unless MFA is completed.

---

## Check Your Understanding

- [ ] Navigate to **Identity** → **Overview** → **Properties** → **Manage security defaults** — confirm the toggle shows **Disabled**
- [ ] Navigate to **Conditional Access** → **Policies** → create a new policy — confirm the **On** state is now available (then discard)
- [ ] From memory: state the three things Security Defaults enforced before you disabled them
- [ ] Navigate to the `CA-Exclusion-BreakGlass` group — confirm both your admin account and the break-glass account are members
- [ ] State why the break-glass account uses the `.onmicrosoft.com` domain rather than the custom domain
- [ ] Explain why you should never disable Security Defaults and leave the tenant in that state without immediately enforcing CA policies

---

## Common Mistakes

**Disabling Security Defaults without creating an emergency access account first.**
You now have no backup if a CA policy locks out all admins. Always create the break-glass account before disabling Security Defaults — not after.

**Using the Authenticator app as the authentication method for the emergency access account.**
The Authenticator app requires a working phone. In an emergency, the phone may be unavailable. The break-glass account must use a method that does not depend on a specific device: a FIDO2 hardware key stored securely, or a long random password that only exists in a physical secure location.

**Not testing the break-glass account after creating it.**
Create the account, then sign in once to verify it works. Verify the password is correct, the MFA method (if any) is working, and the account has the Global Administrator role. A break-glass account you have never tested may fail when you need it most.

**Adding the break-glass exclusion group to some policies but not all.**
If the exclusion group is missing from even one Block policy, an emergency could still lock out your access. Add the exclusion group to every CA policy that uses Block or that requires authentication methods the break-glass account cannot satisfy.

---

## What's Next

Module 4.2 introduces the Conditional Access policy model — the IF-THEN engine. Now that Security Defaults are off, Conditional Access is the only protection on your tenant. Understanding the model fully before building policies is critical — a misunderstood policy creates security gaps or blocks legitimate access. Module 4.2 is the conceptual foundation. The labs start building real policies in Module 4.3.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 3.7 — Secure Your Tenant's Authentication End to End](../../UNIT 3 - AUTHENTICATION/3-7-secure-tenant-authentication-end-to-end/3-7-secure-tenant-authentication-end-to-end.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.2 — What Conditional Access Is →](../4-2-what-conditional-access-is/4-2-what-conditional-access-is.md)
