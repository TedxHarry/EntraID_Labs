# Passwordless Authentication
*Removing the password does not weaken authentication — it eliminates the attack surface that passwords create.*

> 🟡 **Intermediate** · Unit 3, Module 3.5

---

## What You Will Learn

- Explain what passwordless authentication is and how it differs from MFA
- State which passwordless methods are phishing-resistant and which are not
- Configure the Authenticator app for passwordless phone sign-in for Alex Lee
- Walk through the complete passwordless sign-in experience
- Compare sign-in log entries for passwordless vs password-based sign-in

---

## Why This Matters

*SC-300: Implement authentication and access management — Implement and manage passwordless authentication*

Passwords are the root cause of most credential-based attacks — phishing, credential stuffing, brute force, password spray. Passwordless eliminates passwords from the authentication chain. There is no password to phish, no password to spray, and no credential database breach that exposes something useful. Organizations moving to passwordless reduce their attack surface by removing the primary vector attackers use to compromise accounts. SC-300 tests passwordless concepts including the distinction between phishing-resistant and non-phishing-resistant methods.

---

## 🔐 What Passwordless Actually Means

Passwordless authentication means the user proves their identity without typing a password at any point in the sign-in flow. Instead, they use something they have (a device with a private key) plus something they are or something they know at the device level (biometric or PIN) — but never a shared secret transmitted to a server.

This is different from MFA in an important way. MFA typically uses a password as the first factor and a second factor on top. Passwordless replaces the password entirely — the user never types it.

---

## ⚖️ Three Methods — Two Very Different Security Profiles

Microsoft offers three passwordless options. They are not equal.

| Method | How it works | Phishing-resistant? | Where it works |
|---|---|---|---|
| **Microsoft Authenticator (passwordless)** | App prompts; user approves on phone + biometric | No | Any browser sign-in with a phone |
| **Windows Hello for Business** | Biometric or PIN unlocks a private key on the device | Yes | Windows devices joined to Entra ID |
| **FIDO2 security keys** | Hardware key stores a private key; tap to authenticate | Yes | Any platform — Windows, macOS, iOS, Android |

**Why Authenticator passwordless is not phishing-resistant:**
When using the Authenticator app for passwordless sign-in, the authentication still starts with the user entering their email address in a browser. A phishing page can capture that email, relay the sign-in request to Microsoft, and trigger the Authenticator prompt — then convince the user to approve it. The private key never leaves the phone, but the attacker can relay the entire flow.

**Why FIDO2 and Windows Hello are phishing-resistant:**
The private key is bound to a specific origin (the exact website URL). When you sign in at `login.microsoftonline.com`, the key challenge is cryptographically bound to that exact URL. A phishing page at `login.microsoft-365.com` (different URL) cannot use the key — the origin check fails. The key physically cannot respond to a challenge from the wrong site.

> 🔑 **Key concept:** Phishing-resistant authentication means the credential cannot be used on a fake site. Authenticator passwordless is better than a password but is not phishing-resistant. FIDO2 keys and Windows Hello for Business are phishing-resistant. For high-privilege accounts, require phishing-resistant methods.

---

## 📱 How Authenticator Passwordless Sign-In Works

1. User navigates to a sign-in page and enters their email address (but not a password — the page detects that passwordless is configured).
2. Entra ID shows a two-digit number on the screen.
3. The Authenticator app on the user's phone shows a push notification with a list of numbers. The user selects the matching number.
4. The user's biometric (face/fingerprint) or device PIN confirms the selection on the phone.
5. The private key on the phone signs a challenge from Entra ID. Authentication succeeds.

No password was typed. No password was transmitted. The number matching prevents accidental approval of an unexpected prompt.

---

## Lab 3.5.1 — Enable Passwordless Sign-In in the Authentication Methods Policy

**Tools:** Entra admin center
**Prerequisite:** Microsoft Authenticator is enabled for All users (from Module 3.2)

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Protection** → **Authentication methods** → **Policies** → **Microsoft Authenticator**.

3. Select the **Configure** tab.

4. Under **Authentication mode**, change from **Any** (or **Push**) to **Any**. The **Any** mode allows both push-based MFA and passwordless phone sign-in. If it's already set to **Any**, no change is needed.

5. Confirm **Allow passwordless phone sign-in for eligible users** is visible in the configuration. This does not change the target — all users are already enabled. This confirms the Authenticator passwordless sign-in capability is available.

6. Click **Save** if any changes were made.

---

## Lab 3.5.2 — Configure Passwordless Sign-In for Alex Lee

**Tools:** Browser (InPrivate), Microsoft Authenticator app
**Prerequisite:** Microsoft Authenticator is registered for Alex Lee (from Module 3.3). Passwordless mode is enabled in the policy (Lab 3.5.1).

1. Open an InPrivate browser window.

2. Navigate to `aka.ms/mysecurityinfo`. Sign in as **Alex Lee**.

3. On the Security info page, find the **Microsoft Authenticator** entry in the registered methods list.

4. Click the **Change** or **Edit** link next to the Authenticator registration.

5. Look for the option to enable **Passwordless sign-in**. If shown as a toggle or checkbox labeled **Enable phone sign-in**, enable it. Click **Enable** to confirm.

6. A QR code may appear to confirm the configuration between the browser and the Authenticator app. Open the Authenticator app on your phone, tap the **Alex Lee** account entry for the Contoso tenant, and confirm the passwordless setup when prompted.

7. The Authenticator app now shows the account with a **Phone sign-in** indicator instead of the standard one-time code display.

> ⚠️ **Warning:** Enabling passwordless phone sign-in on the Authenticator app disables the traditional TOTP one-time code for that account. If your phone is unavailable, you will need an alternate MFA method to sign in. Ensure a backup method is registered before enabling passwordless.

---

## Lab 3.5.3 — Experience the Passwordless Sign-In Flow

**Tools:** Browser (InPrivate), phone with Authenticator app configured for passwordless
**Prerequisite:** Lab 3.5.2 complete

1. In the InPrivate window, navigate to `portal.office.com` or any Microsoft 365 app.

2. On the sign-in page, enter Alex Lee's UPN: `alex.lee@yourdomain.onmicrosoft.com`.

3. Click **Next**. Instead of a password field appearing, the page shows a two-digit number and a message: "Open Microsoft Authenticator on your phone."

4. On the phone, the Authenticator app shows a push notification listing several numbers. Find the matching number — the same one shown on the browser screen. Tap it.

5. The phone prompts for biometric confirmation (Face ID, fingerprint) or PIN.

6. After confirming, the browser completes authentication. Alex is signed in — without typing a password.

**Compare sign-in log entries:**

7. In your admin browser, navigate to **Identity** → **Monitoring & health** → **Sign-in logs**.

8. Find Alex Lee's most recent sign-in. Click on it to open the detail pane.

9. On the **Authentication details** tab, look at the **Authentication method** column. A passwordless sign-in shows **Microsoft Authenticator** with the authentication detail showing **Phone sign-in** rather than **Push notification MFA**.

10. Compare this to an earlier sign-in where Alex used password + Authenticator push. The method and detail differ — confirming the distinction in the log.

---

## Scenario Check

Alex Lee can now sign in to Microsoft 365 without a password. The sign-in flow requires only a phone with the Authenticator app and a biometric or PIN on that phone. There is no password to type, no password to phish, and no password in the authentication chain.

The sign-in log distinguishes passwordless from password-based sign-ins at the authentication method level. This matters for reporting — when Contoso's security team wants to know how many users are using passwordless, they can filter sign-in logs by authentication method.

Morgan Chen, Jordan Kim, and River Patel are still using password-based authentication. They are next candidates for passwordless enrollment, particularly Jordan Kim — whose IT Administrator role makes a phishing-resistant method more appropriate. In a full deployment, Jordan would use a FIDO2 hardware key rather than the Authenticator app, given the privileged access concern.

---

## Check Your Understanding

- [ ] Sign in as Alex Lee in a new InPrivate window — verify the passwordless flow triggers (no password field appears after entering the email)
- [ ] From memory: name the two Microsoft passwordless options that are phishing-resistant
- [ ] Explain in one sentence why the Authenticator app passwordless method is not phishing-resistant even though no password is used
- [ ] Navigate to **Sign-in logs** — find Alex's most recent sign-in and locate the authentication method shown on the Authentication details tab
- [ ] State what happens to the TOTP code on the Authenticator app when passwordless phone sign-in is enabled
- [ ] For a Global Administrator account — which passwordless method would you recommend and why?

---

## Common Mistakes

**Enrolling all users in Authenticator passwordless and claiming the organization is "phishing-resistant."**
Authenticator passwordless is significantly better than password-based authentication. But phishing-resistant is a specific technical term — the credential is bound to the origin URL and cannot be used on a phishing site. Authenticator passwordless does not meet this bar. Use it for general users. Use FIDO2 or Windows Hello for Business for privileged accounts where phishing-resistance is the explicit requirement.

**Enabling passwordless without a backup method.**
A user whose only authentication method is Authenticator passwordless and who loses or breaks their phone has no way to sign in. Always ensure every user has at least two registered methods — one can be passwordless, the second should be a different method type (backup phone, alternate authenticator account).

**Confusing passwordless with MFA removal.**
Passwordless replaces the password, not the second factor. The biometric or PIN on the device functions as the "something you know or are" factor. The device's private key is the "something you have" factor. Passwordless is still multi-factor — the factors are just different from the traditional password + Authenticator combination.

**Not checking the Conditional Access policy impact.**
If your CA policy requires a "password" authentication context, passwordless sign-in may not satisfy it until the policy is updated to recognize passwordless as a valid authentication strength. In Module 4, CA policies are updated to use Authentication Strength — which is the correct way to specify passwordless requirements in CA.

---

## What's Next

Module 3.6 covers Token Lifetime, Session Management, and Continuous Access Evaluation — what happens to existing access after authentication succeeds. You configured how users sign in. Module 3.6 covers how long that sign-in stays valid, how you can revoke it, and how Entra ID can terminate access in near-real-time when conditions change.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 3.4 — SSPR](../3-4-sspr/3-4-sspr.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.6 — Token Lifetime and Session Management →](../3-6-token-lifetime-session-management-cae/3-6-token-lifetime-session-management-cae.md)
