# Setting Up MFA
*A policy that requires MFA is useless if the user has no MFA method registered.*

> 🟢 **Beginner** · Unit 3, Module 3.3 · ⏱️ 45 minutes

---

## What You Will Learn

- Distinguish between per-user MFA (deprecated) and CA-based MFA (current)
- Register Microsoft Authenticator for Alex Lee through the complete end-user registration flow
- Check MFA registration status across all Contoso users using both the portal and PowerShell
- Identify which users have no MFA method registered
- Issue a Temporary Access Pass (TAP) to onboard a user who cannot scan a QR code

---

## Why This Matters

*SC-300: Implement authentication and access management — Implement and manage multi-factor authentication*

Enabling an authentication method in the policy (Module 3.2) makes it available. A user registering the method is a separate step. Requiring MFA via Conditional Access (Unit 4) is a third step. All three must happen for MFA to work. In production, the most common failure state is: admin enables MFA requirement via CA policy, users have not registered any MFA method, every sign-in is blocked. Checking registration coverage before enforcing a CA policy is a critical step that gets skipped constantly.

---

## 🔄 Per-User MFA vs CA-Based MFA — The Difference That Matters

**Per-user MFA** was the original Microsoft approach. An admin went to each user's account and toggled an MFA state: Disabled → Enabled → Enforced. It was manual, per-user, and applied the same rule to everyone regardless of context.

Per-user MFA is deprecated. Microsoft is removing it. Do not configure it.

**CA-based MFA** is a Conditional Access policy grant control. The policy says: "If these conditions are met, require the user to satisfy MFA." The conditions can be: all users, all apps, all locations. Or: Finance users, SharePoint only, outside trusted networks. CA-based MFA is context-aware, scalable, and the correct approach.

The practical difference: per-user MFA toggles a flag on each user object. CA-based MFA is a policy that runs against every sign-in event and determines whether MFA is required based on the context of that specific sign-in. A user with per-user MFA enabled gets prompted for MFA on every sign-in to every app — always. A user covered by CA-based MFA gets prompted based on the conditions in the policy.

> ⚠️ **Warning:** If per-user MFA is enabled for a user AND a CA policy requires MFA, both requirements are active. The user gets prompted for MFA twice — once by the per-user setting, once by the CA policy. This is a common source of user complaints. Check per-user MFA status before implementing CA-based MFA policies.

---

## 📲 The MFA Registration Flow — What the User Sees

When a user is required to complete MFA for the first time and has no method registered, Entra ID redirects them to the combined security info registration page at `aka.ms/mysecurityinfo`. This page shows all authentication methods available to them based on the Authentication Methods Policy.

The registration process for Microsoft Authenticator:
1. User is redirected to `mysecurityinfo` (or completes a forced registration interrupt during sign-in)
2. User clicks **+ Add method** → **Authenticator app**
3. The page shows a QR code
4. User opens the Microsoft Authenticator app on their phone → **+ Add account** → **Work or school account** → **Scan QR code**
5. After scanning, the app shows a 6-digit code — the user enters it on the page to confirm the registration
6. Registration is complete

When the next CA policy requiring MFA triggers, the user's Authenticator app shows a push notification with a two-digit number (from Module 3.2's number matching setting). The user enters that number in the app. MFA is satisfied.

---

## 🎫 Temporary Access Pass — Onboarding Without a Phone

A Temporary Access Pass (TAP) is a time-limited passcode an admin generates for a specific user. It allows the user to sign in once (or for a limited time) without a registered authentication method — specifically to complete registration.

TAP use cases:
- A new employee whose phone is not yet provisioned
- A user who lost their authenticator device and has no backup
- An admin account that needs to register a FIDO2 key (you can't use a key to register a key — use a TAP first)

A TAP is not an MFA substitute in production. It is a bridge to get a user to a state where they have registered their actual MFA method.

---

## Lab 3.3.1 — Register Microsoft Authenticator for Alex Lee

**Time:** 15 minutes
**Tools:** Browser (InPrivate), Microsoft Authenticator app on a mobile device
**Prerequisite:** Alex Lee exists. Microsoft Authenticator is enabled in the Authentication Methods Policy (Module 3.2). You have the Microsoft Authenticator app installed on a phone.

1. Open an InPrivate or private browser window.

2. Navigate to `aka.ms/mysecurityinfo`.

3. Sign in as **Alex Lee** using Alex's UPN and password `Contoso@1234!`.

4. You may be prompted to complete MFA or a registration interrupt. If prompted, proceed through it.

5. The Security info page opens. Click **+ Add method**.

6. From the dropdown, select **Authenticator app**. Click **Add**.

7. A setup wizard starts. Click **Next** to skip past the download app page (you already have the app).

8. The page shows a QR code.

9. On your phone, open the Microsoft Authenticator app. Tap the **+** button → **Work or school account** → **Scan QR code**.

10. Scan the QR code shown in the browser.

11. After scanning, the app adds an entry for the Contoso tenant. The browser shows a prompt to enter a number for verification. Enter the 6-digit code shown in the Authenticator app for the new account entry.

12. Click **Next**. The registration is confirmed. The page shows "Microsoft Authenticator" as a registered method.

13. Click **Done**.

**Verify the registration in the admin center:**

14. In your regular (admin) browser, navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Authentication methods**.

15. Alex's Microsoft Authenticator registration should appear in the list with a status of **Registered** and the type as **Microsoft Authenticator**.

---

## Lab 3.3.2 — Check MFA Registration Status Across All Contoso Users

**Time:** 15 minutes
**Tools:** Entra admin center, PowerShell
**Prerequisite:** Lab 3.3.1 complete

**Portal method:**

1. In the admin browser, navigate to **Protection** → **Authentication methods** → **User registration details**.

2. This page shows every user in the tenant with columns:
   - **User** — display name and UPN
   - **MFA registered** — Yes or No
   - **MFA capable** — Yes or No (capable means they have a method that can satisfy MFA requirements)
   - **Passwordless capable** — Yes or No
   - **System preferred MFA** — the strongest method the user has registered

3. Review the status for each Contoso user. Alex Lee should now show **MFA registered: Yes**. Morgan Chen, Jordan Kim, River Patel, and Sam Taylor will likely show **No** (they have not registered yet in this lab series).

4. Use the filters at the top to filter by **MFA registered: No**. This shows you the users who need to register before you can enforce an MFA Conditional Access policy.

**PowerShell method:**

5. Open PowerShell and connect:
```powershell
Connect-MgGraph -Scopes "UserAuthenticationMethod.Read.All","User.Read.All"
```

6. Get authentication method registrations for all users:
```powershell
$users = Get-MgUser -All -Property "Id,DisplayName,UserPrincipalName"

foreach ($user in $users) {
    $methods = Get-MgUserAuthenticationMethod -UserId $user.Id
    $methodTypes = $methods | ForEach-Object { $_.AdditionalProperties['@odata.type'] }
    [PSCustomObject]@{
        DisplayName = $user.DisplayName
        UPN = $user.UserPrincipalName
        Methods = ($methodTypes -join ', ')
        HasMFA = if ($methodTypes -notcontains '#microsoft.graph.passwordAuthenticationMethod') {
            "Yes"
        } else {
            if ($methodTypes.Count -gt 1) { "Yes" } else { "No" }
        }
    }
}
```

7. The output shows each user and their registered authentication method types. Users who only have `#microsoft.graph.passwordAuthenticationMethod` have no MFA method beyond their password — they will be blocked when a CA policy requiring MFA is enforced.

---

## Lab 3.3.3 — Issue a Temporary Access Pass for Morgan Chen

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Temporary Access Pass is enabled in the Authentication Methods Policy

> 🔑 **Licensing:** TAP requires **Entra ID P1**. Your developer tenant includes this.

**Enable TAP in the policy (if not already enabled):**

1. Navigate to **Protection** → **Authentication methods** → **Policies** → **Temporary Access Pass**.

2. Set **Enable** to **Yes** and **Target** to **All users**. Click **Save**.

**Issue a TAP for Morgan:**

3. Navigate to **Identity** → **Users** → **All users** → **Morgan Chen** → **Authentication methods**.

4. Click **+ Add authentication method**.

5. From the dropdown, select **Temporary Access Pass**. Click **Add**.

6. A TAP configuration panel opens. Set:
   - **Activate** — leave at current time (activates immediately)
   - **Lifetime (minutes)** — set to `60` (the TAP is valid for 1 hour)
   - **One-time use** — check this box so the TAP can only be used once

7. Click **Create**.

8. The generated TAP is shown on screen. Copy it. This is the only time you will see the full code — it is not stored in retrievable form. In a real scenario, you would give this code to Morgan securely (via phone, chat, or other channel).

9. In an InPrivate browser, navigate to `aka.ms/mysecurityinfo`. Sign in as Morgan Chen using Morgan's UPN. When prompted for a password or MFA, enter the TAP code instead of a password.

10. Morgan is signed in using the TAP. Morgan can now register a proper MFA method (Authenticator app) at the security info page. The TAP is the bridge — not the permanent solution.

---

## Scenario Check

Alex Lee is the first Contoso user with MFA fully registered. The Microsoft Authenticator is linked to Alex's account. When a Conditional Access policy requiring MFA is enforced in Unit 4, Alex will receive a push notification with a two-digit number matching prompt — the experience Alex can expect on every sign-in where MFA is required.

The registration audit showed that Morgan Chen, Jordan Kim, and River Patel have no MFA method registered. Sam Taylor, as a guest, manages their own MFA through their home organization. A TAP was issued for Morgan to allow self-registration. Before enforcing a tenant-wide MFA CA policy in Module 4.3, all internal users need to register — otherwise they will be blocked.

---

## Check Your Understanding

- [ ] Navigate to **Protection** → **Authentication methods** → **User registration details** — confirm Alex Lee shows **MFA registered: Yes**
- [ ] From memory: state the difference between per-user MFA and CA-based MFA in one sentence each
- [ ] State two situations where a Temporary Access Pass is the correct tool to use
- [ ] In the portal, find how many Contoso users currently show **MFA registered: No**
- [ ] Explain what happens when a CA policy requires MFA and the user has no method registered
- [ ] State the purpose of the `aka.ms/mysecurityinfo` URL

---

## Common Mistakes

**Enforcing a CA policy requiring MFA before verifying registration coverage.**
This is the most common MFA deployment mistake. The CA policy is set to Enforce, not Report-Only. Users with no MFA method registered cannot sign in. Helpdesk is flooded with "I can't access anything" tickets. Always check registration via **User registration details** before switching from Report-Only to Enforce.

**Confusing the Authentication Methods Policy with what users have registered.**
Enabling a method in the policy makes it available. It does not register it for users. These are two separate things. Always check both: the policy (what's available) and the registration details (what users have actually set up).

**Issuing a TAP and not treating it as sensitive.**
A TAP allows sign-in without a password. It is a credential. Do not send it via email or leave it in a Teams chat. Deliver it via a secure out-of-band channel and set a short expiration time.

**Leaving per-user MFA enabled while deploying CA-based MFA.**
Per-user MFA causes double MFA prompts when CA policies also require MFA. Check the legacy MFA portal and ensure per-user MFA is set to **Disabled** for all users before enabling CA-based MFA policies.

---

## What's Next

Module 3.4 covers Self-Service Password Reset (SSPR) — enabling users to reset their own passwords without calling the helpdesk. SSPR requires its own registration, separate from MFA registration. The methods available for SSPR are configured in a policy similar to the Authentication Methods Policy. After this module, users can handle both MFA and password resets without admin involvement.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 3.2 — The Authentication Methods Policy](../3-2-authentication-methods-policy/3-2-authentication-methods-policy.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.4 — SSPR →](../3-4-sspr/3-4-sspr.md)
