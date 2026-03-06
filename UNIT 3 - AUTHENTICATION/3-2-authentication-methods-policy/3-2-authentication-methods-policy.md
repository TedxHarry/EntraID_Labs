# The Authentication Methods Policy
*One policy controls what every user in your tenant can use to prove their identity. Get it wrong once, it affects everyone.*

> 🟢 **Beginner** · Unit 3, Module 3.2

---

## What You Will Learn

- Explain what the Authentication Methods Policy is and why it replaced per-user MFA settings
- Navigate to the policy and read what is currently enabled in your tenant
- Enable Microsoft Authenticator for all users with number matching required
- Enable FIDO2 Security Keys
- Restrict SMS to a specific break-glass group rather than all users
- Verify that a user can only register methods that are enabled in the policy

---

## Why This Matters

*SC-300: Implement authentication and access management — Implement and manage authentication methods*

The Authentication Methods Policy is a single panel that controls the authentication method landscape for your entire tenant. If a method is disabled here, no user can register it and no user can use it to sign in — regardless of what any other setting says. Microsoft has been migrating tenants from the legacy per-user MFA portal (which lived at `account.activedirectory.windowsazure.com`) to this centralized policy since 2022. The legacy portal was confusing, per-user, and could not handle modern methods like FIDO2 or Temporary Access Passes. The Authentication Methods Policy is what you use now.

---

## 🔐 What the Policy Controls

The Authentication Methods Policy lives at **Protection → Authentication methods → Policies**. Each method is configured as a separate entry. For each method you can set:

- **Enable/Disable** — whether the method exists in your tenant at all
- **Target** — which users or groups can register and use the method (All users, or specific groups)

The method entries you will work with most:

| Method | What it is | Phishing-resistant? |
|---|---|---|
| **Microsoft Authenticator** | Push notification or passwordless sign-in from the Authenticator app | Push: No. Passkey in Authenticator: Yes |
| **FIDO2 security keys** | Hardware key (YubiKey, Feitian) — tap to sign in | Yes |
| **Temporary Access Pass (TAP)** | Time-limited passcode for onboarding or recovery | N/A — temporary by design |
| **SMS** | One-time code sent to a phone number | No — SIM swap attack vector |
| **Voice call** | One-time code delivered via phone call | No |
| **Certificate-based authentication** | Smart card or certificate-based auth | Yes |
| **Hardware OATH tokens** | Physical token generating time-based codes | No |
| **Software OATH tokens** | OATH TOTP codes from any authenticator app | No |
| **Passkey (FIDO2)** | Passkey stored in Authenticator app or device | Yes |

> 🔑 **Key concept:** SMS is not phishing-resistant. A SIM swap attack or SS7 interception allows an attacker to receive the SMS code without physical access to the phone. SMS should be disabled for all users except break-glass accounts that cannot use app-based methods. This is not a theoretical risk — it is an active attack vector used against corporate accounts.

---

## 📱 Microsoft Authenticator — More Than Push Notifications

When you enable Microsoft Authenticator in the policy, there are sub-settings that matter:

**Authentication mode:**
- **Any** — the user can use the app for MFA (push notification) or passwordless
- **Push** — app-based MFA only (approve/deny prompt)
- **Passwordless** — requires the user to also configure passwordless sign-in

**Number matching (required for push notifications):**
Microsoft started requiring number matching in 2023. Without it, users receive push notifications saying "Approve or Deny?" with no context — an attacker who steals credentials and triggers an MFA prompt can social-engineer a confused user into approving a prompt they didn't initiate (MFA fatigue attack). With number matching enabled, the sign-in page shows a two-digit number the user must type into the Authenticator app before approving. This makes accidental or socially-engineered approvals significantly harder.

Number matching is now the default — Microsoft enabled it tenant-wide. But understanding it and verifying it is configured correctly is part of your job.

**Additional context:**
Shows the application name and sign-in location inside the MFA prompt. Enables users to spot an unexpected authentication from an unfamiliar location.

---

## Lab 3.2.1 — Review the Current Authentication Methods Policy

**Tools:** Entra admin center
**Prerequisite:** None

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. In the left menu, navigate to **Protection** → **Authentication methods** → **Policies**.

3. The policy list opens. You will see all available authentication methods. Note the **Enable** and **Target** columns for each method.

4. Review the current state of these methods in your developer tenant:
   - **Microsoft Authenticator** — is it enabled? Is the target All users?
   - **FIDO2 security keys** — is it enabled?
   - **SMS** — is it enabled? Is it targeting all users?
   - **Temporary Access Pass** — is it enabled?

5. Click on **Microsoft Authenticator** to open its settings panel.

6. In the settings, look for the **Configuration** tab. Check the current state of:
   - **Authentication mode** — Any, Push, or Passwordless
   - **Require number matching** — enabled or not
   - **Show additional context in notifications** — enabled or not

7. Note these current settings without changing them yet.

---

## Lab 3.2.2 — Enable Microsoft Authenticator with Number Matching

**Tools:** Entra admin center
**Prerequisite:** Lab 3.2.1 complete

1. In the Authentication methods policy list, click **Microsoft Authenticator**.

2. At the top of the panel, confirm **Enable** is set to **Yes**. If not, toggle it to **Yes**.

3. Under **Target**, confirm it is set to **All users**. If it is set to a specific group, change it to **All users**.

4. Select the **Configure** tab.

5. Under **Authentication mode**, select **Any** — this allows the method for both MFA (push) and passwordless.

6. Under **Require number matching**, set to **Enabled**. This is the required setting for push-based MFA.

7. Under **Show additional context in notifications**, set to **Enabled**. This shows the app name and location in the push prompt.

8. Click **Save** at the bottom of the panel. You should see a success notification.

---

## Lab 3.2.3 — Enable FIDO2 Security Keys

**Tools:** Entra admin center
**Prerequisite:** Lab 3.2.2 complete

1. Return to the Authentication methods policy list at **Protection** → **Authentication methods** → **Policies**.

2. Click **FIDO2 security keys**.

3. At the top, set **Enable** to **Yes**.

4. Under **Target**, select **All users**.

5. Look at the **Configure** tab. The advanced settings allow you to:
   - **Allow self-service setup** — users can register their own keys at `aka.ms/mysecurityinfo`
   - **Enforce attestation** — require that the key's manufacturer attests to its security (restricts to keys in the FIDO Alliance MDS)
   - **Restrict specific keys** — allow only specific make/model of keys (useful for enterprise hardware procurement)

6. For your developer tenant, leave the default settings and click **Save**.

---

## Lab 3.2.4 — Restrict SMS to a Break-Glass Group

**Tools:** Entra admin center
**Prerequisite:** Lab 3.2.3 complete

First, create a group that will hold break-glass accounts that need SMS fallback.

1. Navigate to **Identity** → **Groups** → **All groups** → **New group**.

2. Create a Security Group named `BreakGlass-SMS` with Assigned membership. Add your Global Administrator account as a member. Click **Create**.

3. Navigate back to **Protection** → **Authentication methods** → **Policies** → **SMS**.

4. If SMS is currently enabled with Target **All users**: change the Target to **Select groups**. Click **Select groups** and choose `BreakGlass-SMS`. Remove **All users** if it appears.

5. If SMS is currently disabled: set **Enable** to **Yes** and set the Target to **BreakGlass-SMS** only.

6. Click **Save**.

7. SMS is now enabled only for accounts in the BreakGlass-SMS group. Standard users (Alex Lee, Morgan Chen, etc.) cannot register SMS as an MFA method. Only the accounts you designate as break-glass have access to SMS fallback.

**Verify scope:**

8. Navigate to **Identity** → **Users** → **All users** → **Alex Lee**.

9. In Alex's profile left menu, select **Authentication methods**. This page shows Alex's currently registered methods. Note that SMS is not listed as a registerable option for Alex — the policy excludes Alex from using SMS.

> ⚠️ **Warning:** Restricting SMS after users have already registered it does not immediately remove their existing SMS registration. Users who registered SMS before the policy change can still use it until they remove it themselves or until an admin deletes the registration. Review existing registrations via **Protection** → **Authentication methods** → **User registration details**.

---

## Scenario Check

The Contoso tenant now has a defined authentication method landscape. Microsoft Authenticator is enabled for all users with number matching required — every push-based MFA prompt will display a two-digit number that the user must enter in the Authenticator app before approving. This eliminates the MFA fatigue attack vector where users approve unexpected prompts.

FIDO2 security keys are enabled for all users — any Contoso employee who obtains a hardware key can register it at `aka.ms/mysecurityinfo` without an admin creating a registration for them.

SMS is restricted to the BreakGlass-SMS group, which contains only the Global Administrator account. Alex Lee, Morgan Chen, Jordan Kim, and River Patel cannot register SMS — they will only be able to use the Authenticator app (or a FIDO2 key if they obtain one). Sam Taylor, as a guest, uses their home organization's authentication methods and is not affected by this policy.

---

## Check Your Understanding

- [ ] Navigate to **Protection** → **Authentication methods** → **Policies** — confirm Microsoft Authenticator shows **All users** and FIDO2 shows **All users**
- [ ] Navigate to **SMS** in the policy — confirm the target is `BreakGlass-SMS` and not **All users**
- [ ] From memory: name two reasons why SMS is not recommended as an MFA method for regular users
- [ ] Explain what number matching does and why it reduces MFA fatigue attacks
- [ ] State what happens to a user's existing SMS registration when you restrict SMS to a specific group after the fact
- [ ] From memory: which two methods in the policy are phishing-resistant?

---

## Common Mistakes

**Disabling SMS globally without first auditing who has SMS registered.**
If users have SMS as their only registered MFA method and you remove their ability to use it, they cannot complete MFA at their next sign-in. Audit existing registrations first using **Protection** → **Authentication methods** → **User registration details**. Remediate users who only have SMS before restricting the method.

**Enabling Authenticator without enabling number matching.**
The default Authenticator push notification (approve/deny) without number matching is vulnerable to MFA fatigue attacks. An attacker with stolen credentials floods the user with approve prompts until the user approves one by mistake. Number matching closes this gap. Always configure number matching when enabling Authenticator.

**Setting target to a group and leaving All users also checked.**
If both a group target and All users are configured, All users wins — the method is available to everyone. Remove All users from the target if you want group-scoped access.

**Confusing the Authentication Methods Policy with per-user MFA settings.**
The legacy per-user MFA settings page still exists at `account.activedirectory.windowsazure.com/usermanagement/mfasettings.aspx`. Microsoft is deprecating it. Settings made there can conflict with the Authentication Methods Policy. If a method appears enabled for a user in the legacy page but is disabled in the policy, the policy wins. Do not use the legacy page for new configuration.

---

## What's Next

Module 3.3 covers MFA registration — how to guide a user through registering methods, how to check registration status across the tenant, and how to handle users who have no MFA method registered. The Authentication Methods Policy from this module controls which methods are available for registration. Module 3.3 is about ensuring every user actually registers one.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 3.1 — From Password to Token](../3-1-from-password-to-token/3-1-from-password-to-token.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.3 — Setting Up MFA →](../3-3-setting-up-mfa/3-3-setting-up-mfa.md)
