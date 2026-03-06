# Creating and Managing Users
*You can create users in the portal and via PowerShell, set every property correctly, and explain why the Object ID matters more than the display name.*

> 🟢 **Beginner** · Unit 2, Module 2.1

---

## What You Will Learn

- Create user accounts in the Entra admin center portal with all required properties set
- Create user accounts using the Microsoft Graph PowerShell SDK
- Explain the difference between cloud-only users and synced users
- State which user properties drive automation and which are purely informational
- Explain why the Object ID is a more reliable identifier than the UPN

*SC-300: Implement and manage user identities — Create, configure, and manage user accounts in Microsoft Entra ID*

## Why This Matters

Every policy, license, group, and governance rule in this series targets a user object. If the user is missing the Usage location field, license assignment fails silently. If the Department field is blank, dynamic group membership does not work. If you use the UPN as a reference identifier in an integration and the user's name changes, the integration breaks. Getting user creation right from the start prevents failures that are hard to trace later. This module creates the five Contoso characters that every remaining lab in the series depends on.

---

## 👤 The User Object — What Entra ID Actually Stores

When you create a user in Entra ID, you are creating an object in the directory. That object has properties. Some properties are required. Some drive automation and policies. Some are purely informational.

**Required properties (the user cannot be created without these):**

| Property | What it is | Example |
|---|---|---|
| Display name | The name shown everywhere in Microsoft 365 | `Alex Lee` |
| User principal name (UPN) | The sign-in name. Format: `username@domain` | `alex.lee@contoso.onmicrosoft.com` |
| Password | Initial password — user changes on first sign-in | Set at creation |

**Properties that drive automation (blank = features break):**

| Property | What it drives | What breaks if blank |
|---|---|---|
| Usage location | License assignment | Cannot assign any Microsoft 365 license |
| Department | Dynamic group membership rules | Dynamic groups filtering by `department` ignore the user |
| Manager | Approval workflows, access reviews, Lifecycle Workflows | Manager-driven approvals fail; no manager shown in My Profile |
| Job title | HR-driven provisioning, informational display | No automation impact in most scenarios |

**Properties that are informational only (no functional impact):**
Street address, city, state, country, office location, phone numbers. These are useful for HR records but do not affect policies, licensing, or group membership.

> 🔑 **Key concept:** Department and Usage location are the two most important properties to set at creation time. Department drives dynamic groups. Usage location is required for licensing. Both are blank by default and easy to forget.

---

## 🆔 Object ID vs UPN: Why the GUID Outlasts the Name

Every user object has two identifiers:

**User Principal Name (UPN)** — the sign-in name in the format `username@domain`. It is human-readable and is used for sign-in. It can be changed. If Alex Lee gets married and changes their name to Alex Johnson, the UPN changes from `alex.lee@contoso.com` to `alex.johnson@contoso.com`.

**Object ID** — a GUID assigned by Entra ID at creation. It never changes, ever. Not when the UPN changes. Not when the display name changes. Not when the user is moved between departments. The Object ID is the permanent identity of that user object in Entra ID.

```
Example Object ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

The Object ID is what Microsoft Graph API uses. It is what appears in audit logs. It is what app integrations use to identify users internally. When you troubleshoot a sign-in log entry or an audit event, the Object ID is what links every record back to the same person.

When you see a user referenced in a script, an API call, or an audit log, use the Object ID — not the display name, not the UPN.

---

## ☁️ Cloud-Only vs Synced Users

The two types of users you will see in any Entra ID tenant:

**Cloud-only users** are created directly in Entra ID — in the portal, via PowerShell, or via the Graph API. Their passwords are stored and validated in Entra ID. All five Contoso characters you create in this module are cloud-only users.

**Synced users** originate in an on-premises Active Directory and are synchronized to Entra ID by Entra Connect Sync or Entra Cloud Sync. They appear in Entra ID but their authoritative source is on-prem AD. Changing a synced user's attributes in Entra ID is overwritten at the next sync cycle.

How to tell the difference: open a user's profile in the admin center and look at the **On-premises sync enabled** field under Properties. Cloud-only users show **No**. Synced users show **Yes** and display a **Source anchor** (the on-prem identifier used to link the two accounts).

You will work with synced users in Unit 10. For the entire series up to Unit 10, all users are cloud-only.

---

## Lab 2.1.1 — Create Alex Lee, Morgan Chen, and Jordan Kim in the Portal

**Tools:** Entra admin center (`entra.microsoft.com`)
**Prerequisite:** Module 1.1 completed. Signed in as your sandbox admin account.

Create these three users exactly as specified. The properties set here are used by labs in Units 3 through 13 — do not skip fields or use different values.

> ⚠️ **Warning:** Replace `yourdomain` in every UPN with your actual sandbox `.onmicrosoft.com` domain (visible on your developer dashboard).

---

**Create Alex Lee**

1. Navigate to **Identity → Users → All users**. Click **+ New user**, then select **Create new user**.

2. On the **Basics** tab, fill in:

   | Field | Value |
   |---|---|
   | User principal name | `alex.lee@yourdomain.onmicrosoft.com` |
   | Display name | `Alex Lee` |
   | Password | Select **Let me create the password**. Set: `Contoso@1234!` |

3. Click the **Properties** tab. Fill in every field below:

   | Field | Value |
   |---|---|
   | First name | `Alex` |
   | Last name | `Lee` |
   | User type | Member |
   | Job title | `Finance Analyst` |
   | Department | `Finance` |
   | Usage location | `United States` |

4. Click **Review + create**, then click **Create**.

   You should see: "Successfully created user Alex Lee."

5. **Verify:** In the search bar, type `Alex Lee` and press Enter. Click **Alex Lee**. Confirm Department shows **Finance** and Usage location shows **United States**. Note the **Object ID** shown on the Overview tab — copy it to your notes.

---

**Create Morgan Chen**

6. Click **+ New user → Create new user**. Fill in:

   **Basics tab:**

   | Field | Value |
   |---|---|
   | User principal name | `morgan.chen@yourdomain.onmicrosoft.com` |
   | Display name | `Morgan Chen` |
   | Password | `Contoso@1234!` |

   **Properties tab:**

   | Field | Value |
   |---|---|
   | First name | `Morgan` |
   | Last name | `Chen` |
   | User type | Member |
   | Job title | `Software Engineer` |
   | Department | `Engineering` |
   | Usage location | `United States` |

7. Click **Review + create**, then **Create**.

---

**Create Jordan Kim**

8. Click **+ New user → Create new user**. Fill in:

   **Basics tab:**

   | Field | Value |
   |---|---|
   | User principal name | `jordan.kim@yourdomain.onmicrosoft.com` |
   | Display name | `Jordan Kim` |
   | Password | `Contoso@1234!` |

   **Properties tab:**

   | Field | Value |
   |---|---|
   | First name | `Jordan` |
   | Last name | `Kim` |
   | User type | Member |
   | Job title | `IT Administrator` |
   | Department | `IT` |
   | Usage location | `United States` |

9. Click **Review + create**, then **Create**.

**Verify all three users:**

10. Navigate to **Identity → Users → All users**. In the search bar, search for `Contoso` — no results, because UPN searches by full match. Instead, use the **Filter** button at the top, select **Department**, and look for Finance, Engineering, and IT. Alternatively, simply scroll and find Alex Lee, Morgan Chen, and Jordan Kim in the list. Confirm all three show **Account status: Enabled**.

**You'll know this worked when** all three users appear in the All users list with Department and Usage location populated.

---

## Lab 2.1.2 — Create Sam Taylor and River Patel Using PowerShell

**Tools:** PowerShell (version 5.1 or 7.x), Microsoft Graph PowerShell SDK
**Prerequisite:** Lab 2.1.1 completed.

> 🔑 **Licensing:** Microsoft Graph PowerShell SDK is free and works with your developer tenant. No additional license required.

**This is the first PowerShell lab in the series. Follow the install steps once — they are not repeated in later modules.**

---

**Step 1 — Install the Microsoft Graph PowerShell SDK**

Open **PowerShell** as Administrator (right-click the Start button, select **Windows PowerShell (Admin)** or **Terminal (Admin)**).

Run:

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Repository PSGallery -Force
```

This downloads the SDK from the PowerShell Gallery. It takes 2–5 minutes. When complete, verify the install:

```powershell
Get-Module Microsoft.Graph -ListAvailable | Select-Object Name, Version | Select-Object -First 1
```

Expected output (version number may differ):
```
Name          Version
----          -------
Microsoft.Graph  2.x.x
```

---

**Step 2 — Connect to Microsoft Graph**

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

A browser window opens prompting you to sign in. Sign in with your **sandbox admin account**. You may see a permissions consent screen — click **Accept**.

When the browser closes, your terminal shows:

```
Welcome to Microsoft Graph!
```

Verify you are connected to the correct tenant:

```powershell
(Get-MgContext).TenantId
```

The output should match the Tenant ID you recorded in Module 1.2.

---

**Step 3 — Create Sam Taylor**

Copy and run the following. Replace `yourdomain` with your actual sandbox domain before running.

```powershell
$passwordProfile = @{
    Password                      = "Contoso@1234!"
    ForceChangePasswordNextSignIn = $false
}

New-MgUser `
    -DisplayName      "Sam Taylor" `
    -UserPrincipalName "sam.taylor@yourdomain.onmicrosoft.com" `
    -GivenName        "Sam" `
    -Surname          "Taylor" `
    -JobTitle         "Contractor" `
    -Department       "Contractors" `
    -UsageLocation    "US" `
    -PasswordProfile  $passwordProfile `
    -AccountEnabled   `
    -MailNickname     "sam.taylor"
```

Expected output (Object ID will differ):
```
Id                                   DisplayName UserPrincipalName
--                                   ----------- -----------------
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx Sam Taylor  sam.taylor@yourdomain.onmicrosoft.com
```

Note the `Id` value — this is Sam's Object ID.

---

**Step 4 — Create River Patel**

```powershell
New-MgUser `
    -DisplayName      "River Patel" `
    -UserPrincipalName "river.patel@yourdomain.onmicrosoft.com" `
    -GivenName        "River" `
    -Surname          "Patel" `
    -JobTitle         "HR Manager" `
    -Department       "HR" `
    -UsageLocation    "US" `
    -PasswordProfile  $passwordProfile `
    -AccountEnabled   `
    -MailNickname     "river.patel"
```

---

**Step 5 — Verify both users were created**

```powershell
Get-MgUser -Filter "Department eq 'Contractors' or Department eq 'HR'" `
    -Property DisplayName, UserPrincipalName, Department, JobTitle, UsageLocation, Id |
    Select-Object DisplayName, UserPrincipalName, Department, JobTitle, UsageLocation, Id
```

Expected output:
```
DisplayName  UserPrincipalName                         Department   JobTitle      UsageLocation Id
-----------  -----------------                         ----------   --------      ------------- --
Sam Taylor   sam.taylor@yourdomain.onmicrosoft.com     Contractors  Contractor    US            xxxxxxxx-...
River Patel  river.patel@yourdomain.onmicrosoft.com    HR           HR Manager    US            xxxxxxxx-...
```

**You'll know this worked when** both users appear in the PowerShell output with Department and UsageLocation populated correctly.

---

## Lab 2.1.3 — Verify All Five Users and Record Their Object IDs

**Tools:** Entra admin center and PowerShell
**Prerequisite:** Labs 2.1.1 and 2.1.2 completed.

All five Contoso characters must be in the tenant and fully verified before proceeding to Module 2.2. Every lab from here through Module 13.4 assumes these five users exist with the exact properties set in this module.

**Portal verification:**

1. Navigate to **Identity → Users → All users**. You should see all five Contoso characters in the list. If any are missing, re-create them now using the steps above.

2. Click each user and verify their profile shows:

   | User | Job title | Department | Usage location |
   |---|---|---|---|
   | Alex Lee | Finance Analyst | Finance | United States |
   | Morgan Chen | Software Engineer | Engineering | United States |
   | Jordan Kim | IT Administrator | IT | United States |
   | Sam Taylor | Contractor | Contractors | United States |
   | River Patel | HR Manager | HR | United States |

3. For each user, copy their **Object ID** from the Overview tab. Record them in a text file — you will reference these IDs in PowerShell commands throughout the series.

**PowerShell verification (reads all five at once):**

```powershell
$users = @("alex.lee", "morgan.chen", "jordan.kim", "sam.taylor", "river.patel")
foreach ($user in $users) {
    Get-MgUser -UserId "$user@yourdomain.onmicrosoft.com" `
        -Property DisplayName, Department, JobTitle, UsageLocation, Id |
        Select-Object DisplayName, Department, JobTitle, UsageLocation, Id
}
```

Expected output — five rows, one per user, all with Department and UsageLocation populated.

**Sign-in log verification:**

4. Open an InPrivate browser window. Sign in as Alex Lee (`alex.lee@yourdomain.onmicrosoft.com` with password `Contoso@1234!`). You will be prompted to set a new password — do so and record it. Sign out.

5. Repeat for River Patel.

6. Return to the admin browser. Navigate to **Identity → Users → Sign-in logs**. Confirm you see sign-in entries for Alex Lee and River Patel. This confirms the accounts are functional, not just created.

> ⚠️ **Warning:** Sam Taylor is a contractor. In a real organization, contractor accounts should be set up as B2B guest users rather than internal member accounts. Module 2.7 covers B2B collaboration and demonstrates the correct approach for external contractors. For now, Sam exists as a member account so they can participate in the group and policy labs in Units 2 through 5.

**You'll know this worked when** all five users appear in the All users list, all properties match the table above, and you have the Object ID for each user recorded.

---

## The Scenario Check

Contoso Ltd now has five identities in Entra ID. Alex Lee is a Finance Analyst in the Finance department — she will have MFA required, Conditional Access policies applied, and be the primary test subject for policy labs throughout the series. Morgan Chen is in Engineering, where the access requirements differ from Finance. Jordan Kim is the IT Administrator who will activate PIM roles in Unit 7. Sam Taylor exists as an internal contractor account for now — Module 2.7 will demonstrate why external contractors should be B2B guests instead. River Patel is the HR Manager who will approve access packages in Unit 8 and trigger lifecycle workflows. All five have Usage location set, which means licenses can be assigned in Module 2.5.

---

## Check Your Understanding

- [ ] Create a sixth test user (`test.admin@yourdomain.onmicrosoft.com`) using PowerShell from memory — include Department, JobTitle, and UsageLocation.
- [ ] Navigate to Alex Lee's profile in the admin center and find her Object ID without reading this module.
- [ ] Run a PowerShell command to retrieve Alex Lee by her Object ID (not her UPN).
- [ ] State which two user properties must be set before a license can be assigned and before dynamic group membership works.
- [ ] Open any user's profile and identify whether they are a cloud-only or synced user. State what field tells you this.
- [ ] Explain in one sentence why the Object ID is more reliable than the UPN as a user identifier.

---

## Common Mistakes

**Forgetting to set Usage location.**
License assignment fails when Usage location is blank, but the error message is not always obvious. The assignment appears to succeed in the portal but the license is never applied. Always set Usage location at creation. If a user cannot receive a license later, check this field first.

**Using different Department values than specified.**
Module 2.4 creates dynamic groups based on the exact `Department` attribute values used here (`Finance`, `Engineering`, `IT`, `Contractors`, `HR`). If you type `finance` (lowercase) or `Finance Dept`, the dynamic group rule `department eq 'Finance'` will not match that user. Use the exact values from the table in Lab 2.1.3.

**Running PowerShell commands without replacing `yourdomain`.**
The commands in Lab 2.1.2 contain the placeholder `yourdomain`. Every command will fail with a "Request_ResourceNotFound" or "Invalid UPN" error if you run them without substituting your actual sandbox domain. Find your domain on the developer dashboard or in **Identity → Overview**.

**Using the deprecated AzureAD or MSOnline PowerShell modules.**
If you have old PowerShell scripts or find examples online that use `New-AzureADUser` or `New-MsolUser`, do not use them. Both modules are deprecated and have been removed from active support. Use only `New-MgUser` from the Microsoft Graph PowerShell SDK.

---

## What's Next

Module 2.2 covers user properties in depth — specifically which properties drive other Entra ID features. You will set up manager relationships (River Patel manages Alex Lee and Morgan Chen), watch a department change propagate to a dynamic group in real time, and see exactly what breaks when Usage location is missing. The users you created in this module are the subjects for every lab in Module 2.2.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 1.4 — How Entra ID Makes Decisions](../../UNIT%201%20-%20GETTING%20STARTED/1-4-how-entra-id-makes-decisions/1-4-how-entra-id-makes-decisions.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.2 — User Properties and What They Drive →](../2-2-user-properties-and-what-they-drive/2-2-user-properties-and-what-they-drive.md)
