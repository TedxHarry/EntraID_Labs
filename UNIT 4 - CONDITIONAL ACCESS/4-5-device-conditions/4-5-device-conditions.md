# Device Conditions — Managed vs Unmanaged
*MFA stops phishing. Device compliance stops token theft. Together, they stop almost everything.*

> 🟢 **Beginner** · Unit 4, Module 4.5 · ⏱️ 50 minutes

---

## What You Will Learn

- The difference between a **compliant device** and a **domain-joined device** in Conditional Access
- How to use device state as a condition in CA policies
- Why device compliance is a stronger control than MFA alone against token theft attacks
- How to require a compliant device for sensitive apps (like Exchange and SharePoint)
- How to verify device compliance status in sign-in logs

---

## Why This Matters

*SC-300: Implement authentication and access management — Configure device-based Conditional Access policies*

Stolen credentials are still the #1 path into corporate networks. MFA stops most phishing attacks — but not token theft. If an attacker steals a user's tokens (through malware on an unmanaged device, a browser extension, or a man-in-the-middle attack), they have valid credentials and valid MFA. They are in.

Device compliance breaks this attack. A device marked as compliant must meet security requirements: disk encryption, strong password, security software, firewall enabled. An attacker on an unmanaged device cannot meet these requirements and cannot pass a device compliance condition — even with valid credentials and MFA. Requiring a compliant device for sensitive resources is the strongest control available short of passwordless authentication.

---

## 📱 Two Device States in CA Conditions

**Compliant device:** The device is enrolled in Intune (Microsoft's Mobile Device Management system) and passes all compliance policies. Compliance policies define rules: disk must be encrypted, password must be at least 14 characters, the device cannot be jailbroken, security software must be active. A Windows 10 PC enrolled in Intune that passes all these rules is a compliant device.

**Hybrid Azure AD joined:** The device is joined to both on-premises Active Directory (AD) and Entra ID. Common in larger enterprises where Windows PCs are domain-joined to AD, and the domain is synchronized to Entra ID. Hybrid Azure AD joined devices can also be marked as compliant via policy.

A device can be:
- **Compliant and Hybrid Azure AD joined** (strongest)
- **Compliant but not hybrid joined** (strong — common for BYOD scenarios with Intune enrollment)
- **Hybrid joined but not compliant** (weaker — device is domain-joined but doesn't meet modern security standards)
- **Neither compliant nor hybrid joined** (no device trust — unmanaged device)

The key point: **Compliance matters more than join status.** A personal laptop enrolled in Intune and compliant is more trustworthy in a modern identity model than a legacy Windows 7 PC that is domain-joined but never updated.

---

## 🔐 Device Compliance vs MFA: Why Device Wins Against Token Theft

**MFA protects the sign-in moment:**
- User types username, password, and provides MFA (code, push, fingerprint)
- MFA challenge is satisfied
- Access token is issued
- User can now access services

**If an attacker steals that token (after sign-in):**
- The attacker has a valid token
- No further MFA challenge is triggered
- The attacker accesses resources as if they were the user
- MFA did not stop this

**Device compliance protects token use:**
- CA policy requires a compliant device
- User signs in on compliant device → device compliance condition passes → tokens issued
- Attacker on unmanaged device tries to use stolen token → token is still valid, but CA re-evaluates when token is used
- If the token is used from an unmanaged device, access is denied (device compliance condition fails)
- The policy blocks token reuse on unauthorized devices

> 🔑 **Key concept:** Device compliance is not about the sign-in. It is about **enforcing which devices can use tokens.** A stolen token is only useful if used on a compliant device. Requiring compliance for sensitive resources (like Exchange and SharePoint) prevents token theft from becoming a breach.

---

## 📋 How CA Evaluates Device State

When a user attempts to access a resource:

1. Is a device compliance condition configured in any policy for this resource?
2. If yes: does the user's current device pass compliance?
3. Compliance check looks up the device in Intune: Is this device enrolled? Does it pass all compliance policies?
4. If yes: device condition passes. If no: policy blocks access.

A CA policy with **Device state → Include: Compliant devices → Grant: Allow** means: only users on compliant devices get access.

A CA policy with **Device state → Include: All devices → Grant: Require MFA** means: all devices (compliant or not) can access, but MFA is required. This policy does not enforce device compliance.

---

## Lab 4.5.1 — Enroll a Test Device in Intune (Simulate Compliance)

**Time:** 20 minutes
**Tools:** Entra admin center, Intune admin center, a Windows PC or Mac (can be the same PC you are using as admin)
**Prerequisite:** Lab 4.1 complete (Security Defaults disabled)

> ⚠️ **Note:** In a real environment, you would enroll a personal device by going to Settings → Accounts → Access work or school → Connect → Sign in with your Entra ID account. For this lab, we will simulate device enrollment in Intune so that your device is marked as compliant. If you already have a device enrolled in Intune, skip to Lab 4.5.2.

**For this lab, we will assume your test device is already enrolled in Intune as compliant.** (In a real M365 developer tenant, you can enroll your own device. For this learning scenario, we will proceed as if enrollment is complete.)

1. Open the Intune admin center at `intune.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Devices** → **All devices**. You should see devices listed. If you see your device, note its name. If not, you are working with an unmanaged device for this lab (which is fine for learning — we will simulate both scenarios).

3. If your device appears, navigate to **Devices** → **Compliance policies** → **+ Create policy**.

4. Set:
   - **Platform:** Windows 10 and later
   - **Name:** `Default Windows Compliance Policy`

5. Under **Device Health:**
   - **Require BitLocker:** Enable
   - **Require Secure Boot to be enabled on the device:** Enable
   - **Require code integrity:** Enable

6. Under **Device Properties:**
   - **Minimum OS version:** 22H2
   - **Maximum OS version:** Leave blank

7. Under **System Security:**
   - **Require passwords to unlock mobile devices:** Enable
   - **Minimum password length:** 14
   - **Password type:** Alphanumeric

8. Click **Next** → **Next** → Assign to **All Devices** → **Create**.

The compliance policy is now defined. Devices enrolled in Intune and meeting these requirements will be marked as compliant. Devices not enrolled or failing these checks will not pass the device compliance condition in CA policies.

---

## Lab 4.5.2 — Build a Device Compliance CA Policy (Report-Only)

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 4.5.1 complete, Lab 4.4 complete (Named Locations)

You will create a policy that requires a compliant device for SharePoint access. This is common in enterprises: SharePoint often contains sensitive documents, so enforcing device compliance adds a strong security layer.

1. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

2. Name the policy: `Require compliant device for SharePoint`

**Users:**

3. Include: **All users**

4. Exclude: **CA-Exclusion-BreakGlass** group

**Target resources:**

5. Include: **Select apps** → Search for and select **Office 365 SharePoint Online**.

6. Click **Select**.

**Conditions:**

7. No conditions needed for this basic lab (but you could add "Locations → Exclude: Contoso-Trusted-Network" to require MFA-only from the office, and device compliance from everywhere else).

**Grant:**

8. Click **Grant** → Select **Require device to be marked as compliant**.

9. For **Require all the selected controls** → toggle to **AND** (not OR). This means the device must pass the compliance check.

10. Do NOT select "Require Hybrid Azure AD joined device" — we are focusing on device compliance alone in this module. (Module 4.8 covers hybrid join scenarios.)

**State:**

11. Set to **Report-only** — DO NOT enforce this yet. Your actual device may not pass all compliance requirements (disk encryption, minimum password length). Running in Report-Only lets you see what the log shows without blocking yourself.

12. Click **Create**.

---

## Lab 4.5.3 — Test the Policy and Read Device Compliance in Sign-In Logs

**Time:** 10 minutes
**Tools:** Entra admin center, Microsoft 365 admin center
**Prerequisite:** Lab 4.5.2 complete

1. Open a new browser tab and navigate to `sharepoint.com`. Sign in as Alex Lee (or another test user).

2. You should see the SharePoint home page. The policy is in Report-Only, so you are not blocked even if your device is not compliant.

3. Now, sign in as yourself (Global Administrator) and navigate to **Protection** → **Conditional Access** → **Sign-in logs**.

4. Search for sign-ins by Alex Lee in the last 1 hour. Click on a sign-in to SharePoint.

5. Under **Conditional Access** section, you will see:
   - `Require compliant device for SharePoint` — Report-only — `Would Block` (if the device is not compliant) or `Satisfied` (if it is)
   - Look at the **Device Compliance Status** field. It will show:
     - ✅ **Compliant** — device passed compliance check
     - ❌ **Non-compliant** — device failed compliance check
     - ⚠️ **Not registered** — device is not enrolled in Intune

6. Make note of the device compliance status shown for this sign-in.

**Understanding the log:**

If the device compliance status is "Non-compliant" or "Not registered," this means:
- The policy is in Report-Only, so the sign-in succeeded
- But if the policy were enforced, the sign-in would be blocked
- The user would need to access from a compliant (Intune-enrolled) device to gain access

If the device compliance status is "Compliant," this means:
- The device passed all compliance checks
- The policy is satisfied
- Even if enforced, this sign-in would be allowed

---

## Lab 4.5.4 — Simulate an Unmanaged Device Sign-In

**Time:** 5 minutes
**Tools:** Browser (incognito mode or separate browser profile)
**Prerequisite:** Lab 4.5.3 complete

To see the full impact of device compliance conditions, sign in from an "unmanaged device."

1. Open an **incognito window** (Ctrl+Shift+N in Chrome, Cmd+Shift+N in Safari, Ctrl+Shift+P in Firefox) or a **new browser profile** that has never signed into Entra ID.

2. In this incognito window, navigate to `sharepoint.com` and sign in as Alex Lee.

3. You will see SharePoint (policy is Report-Only), but the device compliance status in the log will be **Not registered** — because this browser profile has not enrolled any device in Intune.

4. Return to your normal browser and check the sign-in logs again. Filter for the Alex sign-in from the incognito window.

5. Compare the device compliance status:
   - **Normal browser:** Compliant or Non-compliant (based on your enrollment)
   - **Incognito window:** Not registered (unmanaged device)

6. If this policy were enforced, the incognito sign-in would be blocked.

---

## Scenario Check

Contoso now has a device compliance policy in Report-Only for SharePoint. Finance team members (including Alex) frequently access SharePoint for reports and budget documents. The policy requires a compliant device — meaning a device enrolled in Intune with disk encryption, strong password, and security software enabled.

Over the next week, the sign-in logs will show:
- Sign-ins from compliant devices → device compliance condition passes
- Sign-ins from non-compliant or unmanaged devices → device compliance condition fails (but still allowed, since it is Report-Only)

Once the Contoso IT team confirms that the vast majority of Finance team sign-ins are from compliant devices, they will enable the policy to **Enforce**. At that point, any user trying to access SharePoint from an unmanaged device will be blocked.

Jordan Kim (Contoso's security architect) will document this decision: "Compliance policy is enforced for SharePoint because Finance data is sensitive. Users must access from Intune-enrolled devices. For file sync, users can use the Teams desktop app, which has its own App Protection Policy (covered in Unit 9)."

---

## Check Your Understanding

- [ ] Describe the difference between a **compliant device** and a **domain-joined device**. Which is stronger in a modern security model?
- [ ] Explain why MFA alone does not stop token theft, but device compliance does.
- [ ] Navigate to **Sign-in logs** and find a SharePoint sign-in. Identify the **Device Compliance Status** field and state what it shows.
- [ ] Simulate a sign-in from an unmanaged device (incognito) and compare the device compliance status to a sign-in from your normal browser.
- [ ] State the CA logic: Require device to be marked as compliant → what does this policy block?
- [ ] Explain why the device compliance policy for SharePoint should start in Report-Only mode.

---

## Common Mistakes

**Confusing device compliance with device registration.**
Device registration (where a device enrolls in Entra ID) and device compliance (where a device passes Intune compliance policies) are different. A device can be registered but non-compliant, or compliant but not registered. CA policies care about compliance, not just registration.

**Enforcing device compliance before all users have enrolled devices.**
If 20% of your user base has unmanaged devices, enforcing a device compliance policy will block 20% of your users. Always run in Report-Only first to measure how many users would be affected. Then either: (a) give users time to enroll devices, (b) create Exclude groups for users who cannot enroll (contractors, partners), or (c) use a less strict control like MFA instead of device compliance.

**Thinking device compliance is only for Windows PCs.**
iOS and Android devices can also enroll in Intune and be marked compliant. Mac devices also support Intune enrollment (limited feature set). In BYOD scenarios, personal phones enrolled in Intune can be required by CA policies.

**Not understanding that Report-Only shows what would happen.**
Report-Only does not block anything. It logs what the policy would do if it were enforced. A CA policy in Report-Only that shows "Would Block" means: if you enable this policy to Enforce, those sign-ins will be blocked. The log is your test environment.

**Requiring device compliance for all apps from day one.**
Not all apps need device compliance. A strict hierarchy is: low-sensitivity apps (Teams, OneDrive for personal files) require MFA. Medium-sensitivity apps (SharePoint, corporate data) require device compliance. High-sensitivity apps (Azure portal, identity management) require device compliance plus location conditions or even passwordless authentication. Start with sensitive apps, collect data, then expand.

---

## What's Next

Module 4.6 covers **app conditions** in Conditional Access — using CA to scope policies to specific applications. You will learn how to require stronger authentication (phishing-resistant MFA) for the Azure portal, while allowing standard MFA for other apps. Device conditions and app conditions are often combined: require device compliance when accessing sensitive apps from untrusted locations, allow MFA-only from trusted networks.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.4 — Named Locations](../4-4-named-locations/4-4-named-locations.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.6 — App Conditions →](../4-6-app-conditions/4-6-app-conditions.md)
