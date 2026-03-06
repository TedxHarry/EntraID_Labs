# What Is Microsoft Entra ID?
*You now know what Entra ID does, why it exists, and how to navigate the admin center.*

> 🟢 **Beginner** · Unit 1, Module 1.1

---

## What You Will Learn

- Create a free Microsoft 365 E5 developer tenant and access the Entra admin center
- Explain what Microsoft Entra ID is and the problem it was built to solve
- State the difference between Entra ID and on-premises Active Directory
- Identify the three things Entra ID manages: identities, resources, and policies
- Create your first user account and locate their sign-in record in the logs

## Why This Matters

Every task in this series (MFA, Conditional Access, PIM, app integrations, governance) runs through Entra ID. If you do not understand what Entra ID is and why it exists, nothing else in the series will make sense. This module gives you the mental model that everything else builds on.

---

## 🌐 Your Users Moved. Your Identity System Has to Follow.

Ten years ago, everyone worked in an office. Apps ran on servers in a back room. If you controlled the network, you controlled who could access what.

That model is gone. Users work from home, hotels, and coffee shops. Apps live in Microsoft 365, Salesforce, GitHub, and hundreds of other cloud services. There is no corporate network to hide behind.

Microsoft Entra ID is the identity system built for this reality. It controls who people are, what they can access, and under what conditions, across every cloud service and every location.

---

## 🔑 Entra ID Is Not Active Directory in the Cloud

Most people assume Entra ID is "Active Directory in the cloud." It is not.

On-premises Active Directory was built in 1999 for a world of domain controllers, LDAP queries, Kerberos tickets, and users sitting on the office network. It works well for that world.

Entra ID is a different product, designed from the ground up for cloud services and internet-based authentication. It uses different protocols (OAuth 2.0, OpenID Connect, SAML) and different administration tools. The two can coexist — and in many organizations they do — but they are not the same product and you do not manage them the same way.

| | On-premises Active Directory | Microsoft Entra ID |
|---|---|---|
| Designed for | Corporate network, on-prem apps | Cloud services, remote users |
| Authentication protocol | Kerberos, NTLM | OAuth 2.0, OIDC, SAML |
| Admin tool | Active Directory Users and Computers | Entra admin center, Microsoft Graph |
| Works without internet | Yes | No |
| Manages cloud apps natively | No | Yes |

You will connect on-premises AD to Entra ID in Unit 10. For now, your developer tenant is cloud-only.

---

## 🎯 Three Things. That Is All Entra ID Does.

Every feature in this series maps to one of three categories.

**Identities** — Who is allowed to exist in your organization. Users, guests, applications, and devices are all identities in Entra ID. Creating, managing, and protecting these identities is the core admin task.

**Resources** — What those identities can access. Microsoft 365 apps, third-party SaaS applications, Azure services, and custom-built applications can all be connected to Entra ID and protected by it.

**Policies** — The rules that govern when and how access is granted. Conditional Access, MFA requirements, risk-based controls, and device compliance are all policies. They sit between the identity and the resource and decide whether to allow, block, or challenge the sign-in.

Keep this model in mind. When a new concept appears later in the series, ask: is this about an identity, a resource, or a policy?

---

## Lab 1.1.1 — Set Up Your Developer Tenant

**Tools:** Web browser
**Prerequisite:** A personal Microsoft account (Outlook.com or Hotmail.com). Create one free at `account.microsoft.com` if you do not have one.

> ⚠️ **Warning:** This lab provisions your practice environment. Every lab in this series depends on it. Do not skip it.

1. Open your browser and go to `developer.microsoft.com/microsoft-365/dev-program`.

2. Click **Join now**. Sign in with your personal Microsoft account when prompted.

3. On the "Join the Microsoft 365 Developer Program" page, fill in your country and company name. For **Primary focus as a developer**, select **IT Pro**. Accept the terms and click **Next**.

4. On the sandbox setup page, select **Instant sandbox**. Do not select "Configurable sandbox" — the instant sandbox provisions faster and comes pre-loaded with sample users and data.

5. Set a sandbox administrator username and password. Write both down in a secure location. This is your admin account for every lab in this series. Click **Continue**.

6. Microsoft will provision your tenant. This takes 1–5 minutes. When complete, you will see your developer dashboard showing your tenant domain (in the format `yourusername.onmicrosoft.com`) and your admin username.

7. Navigate to `entra.microsoft.com`. Sign in with the sandbox admin account you just created — not your personal Microsoft account.

8. You are now in the Microsoft Entra admin center. You should see the **Home** page with quick-access tiles.

> ⚠️ **Warning:** You now have two Microsoft accounts: your personal account (used to join the developer program) and your sandbox admin account (used for all labs). Always sign in to `entra.microsoft.com` with the sandbox admin account. The most common reason lab steps fail is signing in with the wrong account.

**You'll know this worked when** you can see the Entra admin center home page and the signed-in account in the top-right corner shows your sandbox admin username, not your personal email.

---

## Lab 1.1.2 — Tour the Entra Admin Center

**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Lab 1.1.1 completed. Signed in as your sandbox admin account.

You will use five sections of the admin center constantly throughout this series. Find each one now.

1. In the left navigation menu, expand **Identity**. Click **Users**. You will see a list of users in your tenant. The instant sandbox comes pre-loaded with sample users. Note the columns: **Display name**, **User principal name**, **User type**, **Account status**.

2. Under **Identity**, click **Groups**. You will see sample groups pre-created by the sandbox. Note the **Group type** column — some are **Security**, some are **Microsoft 365**. You will learn the difference in Module 2.3.

3. In the left navigation menu, expand **Applications**. Click **Enterprise applications**. You will see Microsoft services and pre-integrated apps that are connected to your tenant. Every app your organization uses will eventually appear here.

4. In the left navigation menu, expand **Protection**. Click **Conditional Access**. You will see the Conditional Access blade. There are no policies yet. You will build them in Unit 4.

5. Under **Protection**, click **Identity Protection**. Review the dashboard. The reports are empty because there have been no risky sign-ins yet.

6. At the top of the left navigation menu, click the **search icon** (magnifying glass). Type `Sign-in logs` and press Enter. Click **Sign-in logs** in the results. This is where every sign-in to your tenant is recorded. It is currently empty.

7. In the top-right corner, click the **user icon** (your initials or photo). Confirm the signed-in account is your sandbox admin account, not your personal email.

**You'll know this worked when** you can navigate to Users, Groups, Enterprise applications, Conditional Access, and Sign-in logs without using the search bar.

---

## Lab 1.1.3 — Create Your First User and Find Them in the Logs

**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Labs 1.1.1 and 1.1.2 completed.

1. In the left navigation menu, go to **Identity → Users → All users**.

2. Click **+ New user** at the top of the page. Select **Create new user** from the dropdown.

3. On the **Basics** tab, fill in the following:
   - **User principal name:** `testuser01`
   - **Display name:** `Test User 01`
   - Under **Password**, select **Let me create the password** and enter a password you will remember. You will sign in as this user shortly.

4. Click the **Properties** tab. Fill in:
   - **First name:** `Test`
   - **Last name:** `User`
   - **Job title:** `Test Account`
   - **Department:** `IT`
   - **Usage location:** Select your country.

> 🔑 **Key concept:** Usage location is required before you can assign a Microsoft 365 license to a user. If it is blank, license assignment will fail. Set it on every user you create.

5. Click **Review + create**, then click **Create**.

6. You should see a notification: "Successfully created user Test User 01." The user now appears in the **All users** list.

**Verify the user was created:**
In the search bar at the top of the page, type `Test User 01` and press Enter. Click **Test User 01** in the results. On the user's profile page, confirm the display name, UPN, department, and usage location you entered are all showing correctly.

7. Now sign in as the test user to generate a sign-in log entry. Open a new **InPrivate** or **Incognito** browser window — keep your admin window open. Go to `myapps.microsoft.com`.

8. Sign in as `testuser01@yourdomain.onmicrosoft.com`, replacing `yourdomain` with your actual sandbox domain (visible on your developer dashboard). Use the password you set in step 3.

9. You will be prompted to update the password — do so and note the new password. If prompted to set up security info, look for a **Skip** or **Ask later** option and use it.

10. Go back to your admin browser window. Navigate to **Identity → Users → Sign-in logs**.

11. The sign-in from Test User 01 should appear within 1–2 minutes. Click **Refresh** if you do not see it immediately. Find the row for **Test User 01** and click on it.

12. On the sign-in log detail panel, locate and read these fields:
    - **User** — the display name and UPN of who signed in
    - **Application** — which app they signed in to
    - **IP address** — the IP the sign-in came from
    - **Status** — Success or Failure
    - **Conditional Access** — shows "Not applied" because you have no CA policies yet

**You'll know this worked when** you can find the sign-in log entry for Test User 01 and read the Status, Application, and IP address fields without clicking randomly.

---

## The Scenario Check

Contoso Ltd's tenant now exists. The admin environment is running and verified. No Contoso characters are in the tenant yet — Alex Lee, Morgan Chen, Jordan Kim, Sam Taylor, and River Patel are created in Module 2.1. Test User 01 is a placeholder to confirm the environment is working. The sign-in logs are active, which means every action from this point forward is recorded and retrievable.

---

## Check Your Understanding

- [ ] Open `entra.microsoft.com` from memory and navigate to **Sign-in logs** without using the search bar.
- [ ] State the difference between Entra ID and on-premises Active Directory in two sentences.
- [ ] Name the three categories of things Entra ID manages.
- [ ] Find your tenant's primary domain name in the Entra admin center. (Hint: it is in **Identity → Overview**.)
- [ ] Create a second test user (`testuser02`) from memory, including the Usage location field.
- [ ] Find both test users in the **All users** list and confirm both show an account status of **Enabled**.

---

## Common Mistakes

**Signing in with your personal Microsoft account instead of your sandbox admin account.**
The Entra admin center will accept your personal account, but you will see your personal tenant, not your sandbox. If the admin center looks unfamiliar or empty, click the user icon in the top-right corner and check which account is signed in.

**Skipping the Usage location field when creating users.**
Usage location is required before a license can be assigned. If it is blank, license assignment fails without a clear error message. The problem only surfaces later when you try to assign a license and nothing happens. Set it on every user at creation time.

**Selecting "Configurable sandbox" instead of "Instant sandbox" during tenant setup.**
The configurable sandbox requires additional setup steps and does not come with pre-loaded users and data. The instant sandbox is ready in minutes and is what all labs in this series assume.

---

## What's Next

Module 1.2 covers tenants — what they are, what they contain, and why decisions made early are difficult to reverse. You will find your Tenant ID, explore the tenant-level settings available only to Global Administrators, and understand why Microsoft cannot merge two tenants. The developer tenant you created here is your canvas. Module 1.2 explains how that canvas is structured.

---

> 📚 **Entra ID — Zero to Admin** · Unit 1: Getting Started
>
> ← *Start of series* | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 1.2 — Understanding Your Tenant →](../1-2-understanding-your-tenant/1-2-understanding-your-tenant.md)
