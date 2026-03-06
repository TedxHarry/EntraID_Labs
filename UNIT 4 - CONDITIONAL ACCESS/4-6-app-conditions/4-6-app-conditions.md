# App Conditions — Not All Apps Are Equal
*The Azure portal is more dangerous than Teams. Conditional Access should reflect that.*

> 🟢 **Beginner** · Unit 4, Module 4.6

---

## What You Will Learn

- How to use **app conditions** in CA policies to scope policies to specific applications
- The difference between **client app conditions** (what type of client: web, mobile, desktop) and **cloud app conditions** (which service: SharePoint, Teams, Azure portal)
- Why **sensitivity tiers** matter — not all apps need the same controls
- How to require **phishing-resistant MFA** (Authentication Strength) for high-risk apps like the Azure portal
- How to test different policies for different apps and see the difference in user experience

---

## Why This Matters

*SC-300: Implement authentication and access management — Configure app-based Conditional Access policies*

A user logging into Teams to chat with a colleague is a different risk than a user logging into the Azure portal to modify Identity Protection rules. Teams is a communication tool — if compromised, the attacker sees messages. The Azure portal controls infrastructure, billing, and identity configuration — if compromised, an attacker can create rogue admin accounts, delete audit logs, or shut down the entire tenant.

CA policies should reflect this risk difference. A **one-policy-for-everything** approach treats Teams the same as the Azure portal, which is wrong. A **tiered approach** assigns stronger controls to higher-risk apps: standard MFA for general apps, phishing-resistant MFA for admin portals, device compliance for data stores.

App conditions let you write this tiered logic into policy.

---

## 📱 Two Types of App Conditions

**Cloud app conditions:** Which cloud service is the user accessing? Examples:
- Microsoft Graph
- Office 365 SharePoint Online
- Office 365 Exchange Online
- Microsoft Teams
- Azure Portal (Administration)
- Azure Management APIs

You can select **All cloud apps** (broad) or **Select apps** (specific). When you select specific apps, you can include or exclude individual services.

**Client app conditions:** What type of client is being used? Examples:
- **Web browsers** — accessing the app through a web browser (Chrome, Safari, Edge, Firefox)
- **Mobile apps and desktop clients** — Teams app, Outlook app, Intune Company Portal, etc.
- **Exchange ActiveSync clients** — legacy protocol, older Outlook versions, some mobile clients
- **Other clients** — anything else (API access, PowerShell, service principals)

Client app conditions are useful for blocking legacy protocols (Exchange ActiveSync) or requiring stronger auth for legacy Outlook clients while allowing standard MFA for modern Teams.

---

## 🎯 App Sensitivity Tiers

In a mature CA strategy, apps are grouped by sensitivity:

**Tier 1 — Low Sensitivity:**
- Teams, Skype, Yammer
- OneDrive for personal files
- Microsoft 365 apps (Word, Excel online)
- **Requirement:** MFA (standard)

**Tier 2 — Medium Sensitivity:**
- SharePoint (especially where sensitive docs live)
- Exchange (corporate email)
- Project
- **Requirement:** Device compliance OR MFA from trusted network

**Tier 3 — High Sensitivity:**
- Azure Portal
- Azure Management APIs
- Entra Admin Center
- PIM (Privileged Identity Management)
- **Requirement:** Phishing-resistant MFA (Windows Hello, FIDO2 key, app-based conditional approval) + Device compliance for high-privilege operations

**Tier 4 — Critical (not in this module):**
- PIM with elevated access
- Admin activities in audit logs
- **Requirement:** Risk-based dynamic access + MFA + session limitations

Contoso will implement Tiers 1-3 in this module.

---

## 🔐 Authentication Strength — Phishing-Resistant MFA

**Standard MFA (weaker against phishing):**
- SMS codes
- Email codes
- Authenticator app TOTP
- Authenticator app push notification

These are susceptible to phishing: if an attacker tricks a user into signing in on a fake login page and captures their password + SMS code in real time, the attacker is in.

**Phishing-resistant MFA (stronger):**
- Windows Hello for Business (biometric or PIN on device)
- FIDO2 hardware security keys (Yubico, etc.)
- Passwordless sign-in via Microsoft Authenticator (app-based conditional approval)

These cannot be phished because the credential is either:
- Tied to a specific device (Windows Hello) — cannot be used on attacker's device
- A cryptographic key stored on a physical key (FIDO2) — cannot be duplicated
- An approval in an app on the user's phone (Passwordless sign-in) — attacker does not have the phone

For the Azure portal — where a compromised account is catastrophic — requiring phishing-resistant MFA is a high-value control.

> 🔑 **Key concept:** In CA, you do not just require "MFA." You require a specific **authentication strength** (Standard MFA or Phishing-Resistant MFA). The policy enforces which methods are allowed.

---

## 📋 How CA Evaluates Apps

When a user attempts to access a resource:

1. Which cloud app is being accessed? (Teams, SharePoint, Azure Portal, etc.)
2. Is there a CA policy with an **Include: [this app]** condition?
3. If yes: does this policy apply to the user?
4. If yes: evaluate all conditions (location, device, etc.) and grant requirements
5. If all conditions pass and grant requirement is met → allow
6. If any condition fails or grant requirement not met → block or challenge

A policy with **Cloud apps → Include: All cloud apps** applies to every service. A policy with **Cloud apps → Include: Selected apps → Azure Portal** applies only when accessing the Azure Portal.

---

## Lab 4.6.1 — Understand App Selection in CA Policy

**Tools:** Entra admin center
**Prerequisite:** Lab 4.1 complete (Security Defaults disabled)

This lab is exploratory — no policy is created yet. You will navigate the CA interface and see what apps are available to select.

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

3. Under **Target resources**, click **0 resources selected**.

4. Click the **Cloud apps** tab (if not already selected).

5. You will see three options:
   - **All cloud apps** (includes all Microsoft cloud services and third-party SaaS)
   - **Select apps** (pick specific ones)
   - **Exclude apps** (all except these)

6. Click **Select apps**. A search box appears. Start typing app names and observe:
   - Type `azure` → see **Azure Portal**, **Azure Management**, **Azure Data Lake**, etc.
   - Type `sharepoint` → see **Office 365 SharePoint Online**
   - Type `teams` → see **Microsoft Teams**
   - Type `exchange` → see **Office 365 Exchange Online**

7. Do NOT create a policy yet. Just observe the list of available apps.

8. Click **Cancel** to close this dialog.

**Key observation:** Entra ID has a built-in list of **cloud apps** that you can reference. When building a CA policy, you select which apps the policy applies to. Different policies can apply different controls to different apps.

---

## Lab 4.6.2 — Create a Tier 3 Policy: Azure Portal Requires Phishing-Resistant MFA

**Tools:** Entra admin center
**Prerequisite:** Lab 4.6.1 complete

You will create a policy that requires **phishing-resistant MFA** (Authentication Strength) for anyone accessing the Azure Portal or Azure Management APIs. This is a Tier 3 (high sensitivity) app.

### Step A: Create an Authentication Strength Policy (if not already exists)

First, you must define what "phishing-resistant MFA" means in your organization. You do this via **Authentication Strength**.

1. Navigate to **Protection** → **Authentication methods** → **Authentication strengths**.

2. Click **+ New authentication strength policy**.

3. Configure:
   - **Name:** `Phishing-resistant MFA`
   - **Description:** Windows Hello, FIDO2 keys, or passwordless sign-in only — no SMS or email codes

4. Under **Allowed methods**, uncheck all methods except:
   - ✅ Windows Hello for Business
   - ✅ FIDO2 security key
   - ✅ Microsoft Authenticator (passwordless phone sign-in)

5. Uncheck all others (SMS, email, TOTP, push notification).

6. Click **Next** → **Create**.

You now have an **Authentication Strength** policy named "Phishing-resistant MFA" that restricts which MFA methods are allowed.

### Step B: Create the CA Policy for Azure Portal

7. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

8. Name: `Require phishing-resistant MFA for Azure Portal`

**Users:**

9. Include: **All users**

10. Exclude: **CA-Exclusion-BreakGlass** group

**Target resources:**

11. Click **0 resources selected** → **Cloud apps** → **Select apps**.

12. Search for `Azure`. Select:
    - **Azure Portal**
    - **Azure Management** (this covers ARM APIs and other management endpoints)

13. Click **Select** and close.

**Conditions:**

14. No conditions required for this basic lab. (In production, you might exclude trusted networks or require device compliance as well.)

**Grant:**

15. Click **Grant** → Select **Require authentication strength**.

16. From the dropdown, select **Phishing-resistant MFA** (the policy you created in Step A).

17. Leave **Require all the selected controls** as **AND**.

**Session:**

18. No session controls needed for this lab.

**State:**

19. Set to **Report-only** for now. DO NOT enable to Enforce yet — you need to verify the experience first.

20. Click **Create**.

---

## Lab 4.6.3 — Create a Tier 1 Policy: Other Apps Require Standard MFA

**Tools:** Entra admin center
**Prerequisite:** Lab 4.6.2 complete

Now you will create a second policy: for all other apps (Teams, SharePoint, Exchange, OneDrive), require standard MFA. This is simpler than the Azure Portal policy.

1. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

2. Name: `Require MFA for all other apps`

**Users:**

3. Include: **All users**

4. Exclude: **CA-Exclusion-BreakGlass** group

**Target resources:**

5. Click **0 resources selected** → **Cloud apps** → **Select apps**.

6. Search for and select:
   - **Microsoft Teams**
   - **Office 365 SharePoint Online**
   - **Office 365 Exchange Online**
   - **OneDrive for Business**

7. Click **Select** and close.

**Conditions:**

8. No conditions.

**Grant:**

9. Click **Grant** → Select **Require multifactor authentication**.

10. Leave **Require all the selected controls** as **AND**.

**State:**

11. Set to **Report-only**.

12. Click **Create**.

You now have two CA policies:
- **Azure Portal requires phishing-resistant MFA** (Tier 3)
- **Other apps require standard MFA** (Tier 1)

---

## Lab 4.6.4 — Test Different Apps and Observe User Experience

**Tools:** Entra admin center, browser
**Prerequisite:** Lab 4.6.3 complete

Now you will test the different policies by signing in to different apps and observing the different MFA challenges.

### Test 1: Sign In to Teams (Standard MFA)

1. Open a new browser tab and navigate to `teams.microsoft.com`.

2. Sign in as Alex Lee.

3. You will be challenged for **standard MFA**. The Authenticator app will send a push notification (or you will see a code prompt). Approve or enter the code.

4. You are now in Teams. Note which MFA method was required.

### Test 2: Sign In to Azure Portal (Phishing-Resistant MFA)

5. Open a new browser tab and navigate to `portal.azure.com`.

6. Sign in as Alex Lee.

7. **Observation:**
   - **If you have Windows Hello configured:** You will be challenged for Windows Hello (biometric or PIN). This is phishing-resistant.
   - **If you do NOT have Windows Hello:** You will see an error message saying "This sign-in doesn't meet authentication requirements" because Alex does not have any phishing-resistant MFA method configured.

8. If you see the error, this is expected and correct. The policy is working: it is requiring phishing-resistant MFA, which Alex does not have. In a real scenario, Alex would need to enroll in Windows Hello or a FIDO2 key first.

### Test 3: Check Sign-In Logs

9. Return to the Entra admin center. Navigate to **Protection** → **Conditional Access** → **Sign-in logs**.

10. Filter for Alex Lee's sign-ins in the last 10 minutes. You should see:
    - **Teams sign-in:** MFA method = `Authenticator app push` (or whatever standard MFA Alex used)
    - **Azure Portal sign-in:** CA result = `Would Block` (in Report-Only, so not actually blocked), required authentication strength = `Phishing-resistant MFA`

11. Click on the Azure Portal sign-in to see details:
    - **Conditional Access** section shows: `Require phishing-resistant MFA for Azure Portal — Report-only — Would Block`
    - **Authentication Details** shows: Alex's MFA method is `Authenticator app push`, which does NOT satisfy phishing-resistant MFA requirement
    - So the policy would block this sign-in if enforced

12. Click on the Teams sign-in:
    - **Conditional Access** section shows: `Require MFA for all other apps — Report-only — Satisfied`
    - The standard MFA method (Authenticator app) is sufficient

---

## Lab 4.6.5 — Use What-If Tool to Simulate Different Scenarios

**Tools:** Entra admin center
**Prerequisite:** Lab 4.6.4 complete

The What-If tool lets you simulate policy impact without actually signing in. This is useful for testing before enforcing.

1. Navigate to **Protection** → **Conditional Access** → **What If**.

2. Scenario 1: **Standard user, Teams, standard MFA**
   - User: **Alex Lee**
   - App: **Microsoft Teams**
   - Compliance state: Leave at default (not required for this scenario)
   - Click **What If**
   - Result: Should show `Require MFA for all other apps — Satisfied` (standard MFA requirement is met)

3. Scenario 2: **Standard user, Azure Portal, standard MFA**
   - User: **Alex Lee**
   - App: **Azure Portal**
   - Compliance state: Leave at default
   - Click **What If**
   - Result: Should show `Require phishing-resistant MFA for Azure Portal — Would Block` (Alex does not have phishing-resistant MFA)

4. Scenario 3: **Break-glass account (should be excluded), Azure Portal**
   - User: **Emergency Access Account** (or your break-glass admin)
   - App: **Azure Portal**
   - Click **What If**
   - Result: Should show `Require phishing-resistant MFA for Azure Portal — Not Applied` (because this user is in the exclusion group)

5. Observe the What-If results for each scenario. This is how you verify policy logic before enforcing.

---

## Scenario Check

Contoso now has app-based CA policies in place:

**For the Azure Portal and Azure Management APIs:**
- Requirement: Phishing-resistant MFA (Windows Hello, FIDO2, or Passwordless sign-in)
- Status: Report-Only (collecting data)
- Impact: High-privilege access is protected against phishing attacks. Only users with phishing-resistant MFA configured can access Azure portal.

**For Teams, SharePoint, Exchange, OneDrive:**
- Requirement: Standard MFA
- Status: Report-Only
- Impact: General apps require MFA, but standard methods (Authenticator app, SMS) are accepted

Jordan Kim (security architect) documents:
> "We have separated app policies by sensitivity. Azure Portal is Tier 3 and requires phishing-resistant MFA because infrastructure access is critical. General collaboration apps (Teams, SharePoint) are Tier 1 and require standard MFA. Over the next week, the report-only logs will tell us how many users would be blocked by the Azure Portal policy (we need to enroll them in Windows Hello or FIDO2 first before enforcing). Teams and SharePoint policies should have minimal impact since standard MFA adoption is high."

---

## Check Your Understanding

- [ ] Explain the difference between a **cloud app condition** and a **client app condition**. Give examples of each.
- [ ] Name three apps that would be classified as Tier 3 (high sensitivity). Explain why each one is high sensitivity.
- [ ] State the difference between **standard MFA** and **phishing-resistant MFA**. Why would you require phishing-resistant MFA for Azure Portal?
- [ ] In the What-If tool, simulate a sign-in to the Azure Portal by Alex Lee. What does the result show? Why?
- [ ] Explain the logic: Create one policy for Azure Portal (phishing-resistant MFA) and a different policy for Teams (standard MFA). How would a user experience this differently?
- [ ] You enforce the Azure Portal phishing-resistant MFA policy without enrolling users in Windows Hello first. What happens? Is this a mistake?

---

## Common Mistakes

**Requiring phishing-resistant MFA before users have it configured.**
Windows Hello, FIDO2 keys, and Passwordless sign-in require prior setup. If you enforce a phishing-resistant MFA policy before users are enrolled, they will be locked out. Always run in Report-Only first, measure how many users have phishing-resistant methods configured, then give the rest time to enroll before enforcing. For the Azure Portal, you may need a 2-4 week rollout.

**Using "All cloud apps" when you should use "Select apps".**
A policy with "All cloud apps" applies to every service — including Teams, OneDrive, third-party apps integrated with Microsoft Graph, and more. This is broad and often unintended. Use "Select apps" to target specific high-value services. Build your Tier 3 (Azure Portal) policy first, then expand to Tier 2 (SharePoint) if needed.

**Not excluding the Break-Glass admin account.**
The break-glass emergency account (often a Cloud-only Global Administrator) must be excluded from all CA policies. If the break-glass account is subject to a phishing-resistant MFA requirement and does not have that method configured, you cannot use it to regain access in an emergency. Always verify that CA-Exclusion-BreakGlass is excluded from every CA policy.

**Forgetting that app conditions stack.**
If you have two policies — one for "All cloud apps" requiring MFA, and one for "Azure Portal" requiring phishing-resistant MFA — both apply when accessing Azure Portal. The more restrictive policy (phishing-resistant MFA) wins. The user must satisfy the stricter requirement. This is by design (defense in depth), but it surprises new admins.

**Confusing authentication strength with MFA requirement.**
`Require multifactor authentication` means "MFA is required" but does not specify which methods. Any MFA method (SMS, email, Authenticator app, Windows Hello, etc.) satisfies it.

`Require authentication strength: Phishing-resistant MFA` means "MFA is required AND it must be one of these specific methods" (Windows Hello, FIDO2, Passwordless sign-in). It is stricter. Always use authentication strength for high-sensitivity apps.

---

## What's Next

Module 4.7 covers **Report-Only mode and testing safely** — how to use Report-Only CA policies to collect data about your environment before enforcing changes. You will learn how to read report-only results in sign-in logs and use data to inform rollout decisions. By then, you will have multiple CA policies in Report-Only (from modules 4.3, 4.4, 4.5, and 4.6) — module 4.7 teaches you how to manage and enable them carefully.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.5 — Device Conditions](../4-5-device-conditions/4-5-device-conditions.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.7 — Report-Only Mode and Testing →](../4-7-report-only-mode-and-testing/4-7-report-only-mode-and-testing.md)
