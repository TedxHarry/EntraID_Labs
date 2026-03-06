# SSPR — Self-Service Password Reset
*Every password reset ticket that reaches the helpdesk is a failure of configuration.*

> 🟢 **Beginner** · Unit 3, Module 3.4

---

## What You Will Learn

- Enable SSPR for all users in the Contoso tenant
- Configure the required number of authentication methods for reset
- Register SSPR methods as Alex Lee at `aka.ms/ssprsetup`
- Complete a self-service password reset at `aka.ms/sspr`
- Locate the SSPR event in the audit log and read the result

---

## Why This Matters

*SC-300: Implement authentication and access management — Implement and manage self-service password reset*

The average password reset ticket costs an organization between $15 and $70 in helpdesk labor. In a 50-person company with predictable password forgetting rates, that adds up fast. SSPR eliminates the ticket for users who have registered their reset methods. In practice, SSPR adoption depends on two things: enabling it and making sure users register before they're locked out. An admin who enables SSPR after the fact is too late for the user who already can't sign in. Proactive registration — before the lockout — is the part most organizations get wrong.

---

## 🔑 What SSPR Is and Is Not

SSPR allows a user who cannot sign in — because they forgot their password or their account is locked — to prove their identity through alternative methods and reset their own password without contacting IT.

SSPR is **not** the same as MFA registration. They are separate registration flows, separate method configurations, and separate portals. A user who has registered Microsoft Authenticator for MFA has not automatically registered for SSPR. They need to register separately at `aka.ms/ssprsetup`.

SSPR is **not** available for accounts synced from on-premises Active Directory unless **password writeback** is configured. For cloud-only users (like all Contoso users), SSPR resets the Entra ID password directly. For synced users, SSPR must write the new password back to the on-premises AD. Without writeback, the reset appears to succeed but the next on-premises sign-in still uses the old password.

---

## ⚙️ SSPR Configuration Options

SSPR is configured at **Protection** → **Password reset**. The key settings:

**Who can use SSPR:**
- **None** — SSPR is disabled
- **Selected** — specific groups only
- **All** — all users in the tenant

**Authentication methods (for SSPR):**
The methods users can use to verify their identity during a reset:

| Method | What it requires |
|---|---|
| Mobile app notification | Approve a prompt in Microsoft Authenticator |
| Mobile app code | Enter a code from the Authenticator app |
| Email | Receive a code at an alternate external email address |
| Mobile phone | Receive an SMS or voice call |
| Office phone | Receive a voice call at an office number |
| Security questions | Answer 3–5 preset questions |

**Number of methods required:**
- **1** — user must verify identity with one method
- **2** — user must verify with two different methods (more secure, reduces social engineering risk)

The recommended setting is 2 methods. This means an attacker who knows the user's alternate email address still can't reset the password without also having the phone or the authenticator app.

---

## 🖥️ Two URLs to Know

| URL | What it does |
|---|---|
| `aka.ms/ssprsetup` | SSPR registration — users register their reset methods |
| `aka.ms/sspr` | SSPR reset — users reset their password when locked out |

These are different from the MFA registration URL (`aka.ms/mysecurityinfo`). The security info page does cover both MFA and SSPR registration in a combined flow, but the dedicated SSPR setup URL is what to direct users to for SSPR-specific registration.

---

## Lab 3.4.1 — Enable and Configure SSPR

**Tools:** Entra admin center
**Prerequisite:** None

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Protection** → **Password reset**.

3. On the **Properties** page, under **Self service password reset enabled**, select **All**. Click **Save**.

4. Select the **Authentication methods** page from the left menu (within the Password reset section — not the main Authentication methods section).

5. Under **Number of methods required to reset**, change the value to `2`.

6. Under **Methods available to users**, ensure these are checked:
   - **Mobile app notification**
   - **Mobile app code**
   - **Email**
   - **Mobile phone (SMS only)** — optional, acceptable for SSPR unlike MFA

7. Uncheck **Security questions** — answers to security questions are guessable through social engineering. They are not recommended.

8. Click **Save**.

---

## Lab 3.4.2 — Register SSPR Methods as Alex Lee

**Tools:** Browser (InPrivate), phone with Microsoft Authenticator
**Prerequisite:** Lab 3.4.1 complete. Microsoft Authenticator is registered on Alex's account from Module 3.3.

1. Open an InPrivate browser window.

2. Navigate to `aka.ms/ssprsetup`.

3. Sign in as **Alex Lee** (UPN and password `Contoso@1234!`). Complete the MFA prompt using the Authenticator app registered in Module 3.3.

4. The SSPR setup page opens. It shows methods Alex can register for password reset.

5. Click **Set it up now** next to **Authentication app**. The Authenticator app is already registered for MFA — SSPR can reuse it. Click **Confirm** to approve using it for SSPR as well.

6. Now register a second method. Click **Set it up now** next to **Email**. Enter an external email address you control (not Alex's work email — this must be a personal email or backup email). Enter the code sent to that address. Click **Verify**.

7. Both methods are now registered. The SSPR setup page shows two methods registered — the Authenticator app and the backup email.

8. Click **Looks good**.

---

## Lab 3.4.3 — Test SSPR as Alex Lee

**Tools:** Browser (InPrivate)
**Prerequisite:** Lab 3.4.2 complete

1. In the InPrivate browser, navigate to `aka.ms/sspr`.

2. Enter Alex's UPN: `alex.lee@yourdomain.onmicrosoft.com`. Complete the CAPTCHA challenge.

3. Click **Next**.

4. SSPR prompts Alex to verify identity using one of the registered methods. Select **Approve a notification in my Authenticator app**. Click **Send notification**.

5. On the phone, the Authenticator app receives a notification. Open it and approve. The SSPR flow advances to step 2 (if 2 methods required) or directly to the reset page.

6. For the second method (if 2 required): select **Email my alternate email**. Enter the alternate email you registered. Click **Send email**.

7. Open the alternate email inbox. Find the email from Microsoft. Enter the verification code shown in the email.

8. SSPR shows the password reset page. Enter a new password for Alex: `Contoso@5678!`. Confirm the new password.

9. Click **Finish**. The password is reset. Alex can now sign in with `Contoso@5678!`.

> ⚠️ **Warning:** After testing SSPR, reset Alex's password back to the original `Contoso@1234!` so subsequent labs work as expected. Navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Reset password** as your admin account and set a new password, or sign in as Alex and change it again via SSPR.

**Verify the SSPR event in the audit log:**

10. In your admin browser, navigate to **Identity** → **Monitoring & health** → **Audit logs**.

11. Filter by **Service: Self-service password management** or search for Alex Lee in the filter.

12. Find the **Reset password (self-service)** entry. Click on it. The detail pane shows:
    - **Initiated by:** Alex Lee (the user self-initiated, not an admin)
    - **Result:** Success
    - **Target:** Alex Lee
    - **Activity details:** The methods used to verify identity

---

## Scenario Check

Contoso now has SSPR enabled for all users with a 2-method verification requirement. Alex Lee successfully reset their own password without contacting Jordan Kim or any helpdesk. The audit log recorded the event as a self-service reset — initiated by the user, no admin involvement.

River Patel and Morgan Chen have not registered for SSPR yet. If either of them forgets their password before registering, they will need admin intervention. The recommended approach: send all users a proactive email directing them to `aka.ms/ssprsetup` before the next quarterly password expiration — if password expiration is configured.

When Contoso implements full lifecycle automation in Module 8, new hire onboarding will include a forced registration interrupt that sends new users to the combined security info page. Users will register both MFA and SSPR methods before completing their first sign-in.

---

## Check Your Understanding

- [ ] Navigate to **Protection** → **Password reset** → **Properties** — confirm SSPR is set to **All**
- [ ] Navigate to **Authentication methods** within Password reset — confirm 2 methods are required and Security questions is unchecked
- [ ] From memory: state the difference between `aka.ms/ssprsetup` and `aka.ms/sspr`
- [ ] Navigate to **Audit logs** and find Alex's SSPR event — state who initiated it (should be Alex Lee, not an admin)
- [ ] Explain why SSPR and MFA registration are separate flows that a user must complete separately
- [ ] State what happens when a hybrid-synced user tries to use SSPR without password writeback configured

---

## Common Mistakes

**Enabling SSPR after users are already locked out.**
The user who forgot their password and cannot sign in cannot navigate to `aka.ms/ssprsetup` to register. SSPR registration must happen before the lockout. The correct deployment order: enable SSPR, communicate the registration URL to all users, give them 2 weeks to register, then monitor registration coverage.

**Setting required methods to 1 for convenience.**
A single-method SSPR verification is vulnerable to social engineering. An attacker who knows the user's alternate email can intercept the reset code if they also compromised the email account. Two methods significantly raise the difficulty. Set it to 2.

**Assuming Authenticator MFA registration covers SSPR.**
It does not. MFA registration (Module 3.3) and SSPR registration are separate. A user who has the Authenticator app for MFA must still explicitly register for SSPR at `aka.ms/ssprsetup`. The security info page (`aka.ms/mysecurityinfo`) combines both, but the user must still complete the SSPR registration step within it.

**Not testing SSPR after enabling it.**
SSPR involves email delivery, app push notifications, and phone numbers — all of which can fail silently. Test the complete reset flow from a non-admin account before telling users it is available. An SSPR configuration that appears correct in the portal but fails at the verification step leaves users unable to recover.

---

## What's Next

Module 3.5 covers passwordless authentication — eliminating the password from the sign-in flow entirely. The Authenticator app registered in Module 3.3 can be configured for passwordless sign-in. FIDO2 hardware keys enabled in Module 3.2 are the most phishing-resistant passwordless option. Module 3.5 walks through both and explains which scenarios each is suited for.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 3.3 — Setting Up MFA](../3-3-setting-up-mfa/3-3-setting-up-mfa.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.5 — Passwordless Authentication →](../3-5-passwordless-authentication/3-5-passwordless-authentication.md)
