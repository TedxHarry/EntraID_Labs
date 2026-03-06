# Bulk Operations — Managing Users at Scale
*One portal action per user works for five users. It doesn't work for fifty.*

> 🟢 **Beginner** · Unit 2, Module 2.6

---

## What You Will Learn

- Create multiple users in a single operation using a CSV file upload in the portal
- Perform bulk updates on a set of users using PowerShell
- Disable a group of users with a single PowerShell command
- Delete and restore users from the 30-day soft-delete recycle bin
- Write a PowerShell script that creates users from a CSV and sets all required properties

---

## Why This Matters

*SC-300: Implement and manage user identities — Create and manage bulk user operations in Microsoft Entra ID*

When Contoso acquires another company or hires a 20-person department, individual user creation is not viable. The portal's bulk user creation accepts a CSV file and processes it in a background job. PowerShell handles more complex scenarios: setting properties not available in the bulk upload, conditional logic, output logging. Both methods are on the SC-300 exam. You will also encounter the soft-delete recycle bin — something every admin needs to know when a user was deleted by mistake.

---

## 📋 Three Methods for Working at Scale

| Method | Best for | Limitations |
|---|---|---|
| Portal CSV upload | Quick creation of many users with standard properties | Limited property set; no conditional logic |
| PowerShell script | Full control over all properties; can read from any data source | Requires scripting knowledge |
| Graph API (Invoke-MgGraphRequest) | Automation pipelines and integrations | Requires API knowledge; covered in Unit 11 |

The CSV upload is the fastest starting point. PowerShell handles everything the CSV can't.

---

## 📁 The CSV Upload Format

The Entra admin center accepts a specific CSV format for bulk user creation. You must use the template — a custom format will fail.

The template has these required columns:

| Column | Required | Notes |
|---|---|---|
| `Name [displayName] Required` | Yes | Full display name |
| `User name [userPrincipalName] Required` | Yes | Must be `username@yourdomain.onmicrosoft.com` |
| `Initial password [passwordProfile] Required` | Yes | Must meet password complexity |
| `Block sign in (Yes/No) [accountEnabled] Required` | Yes | `No` = account enabled |
| `First name [givenName]` | No | |
| `Last name [surname]` | No | |
| `Job title [jobTitle]` | No | |
| `Department [department]` | No | |
| `Usage location [usageLocation]` | No | Two-letter country code: `US`, `GB`, `DE` |

The template is downloaded directly from the portal — do not manually build the column structure. The header row formatting is exact and the upload fails if any required column header is altered.

---

## ♻️ The 30-Day Soft Delete

When you delete a user in Entra ID, the account is not immediately destroyed. It moves to the **Deleted users** bin and stays there for 30 days. During those 30 days:

- The user cannot sign in
- Their licenses are released back to the tenant pool
- Their Object ID is preserved
- You can restore them to full active status with one click

After 30 days, Entra ID permanently deletes the account and all associated data. There is no recovery after permanent deletion.

The **Deleted users** bin is at `Identity → Users → Deleted users`. Restore is a single button on the deleted user's page.

> ⚠️ **Warning:** When you restore a deleted user, their licenses are not automatically restored. The account comes back active, but if the license was released during the 30-day period, it must be re-assigned. Check the user's Licenses page after every restore.

---

## Lab 2.6.1 — Bulk Create Users via CSV Upload

**Tools:** Entra admin center
**Prerequisite:** You know your tenant's `.onmicrosoft.com` domain name (from Module 1.2)

**Step 1: Download the CSV template**

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Identity** → **Users** → **All users**.

3. Click **Bulk operations** at the top of the page, then select **Bulk create**.

4. The Bulk create users panel opens. Click **Download** to get the CSV template. The file downloads as `bulkCreateTemplate.csv`.

5. Open the file in Excel or any text editor. Do not change the header row. You will fill in data below it.

**Step 2: Fill in the CSV**

6. Add 10 test users below the header. Here is the data to use — replace `yourdomain` with your actual `.onmicrosoft.com` domain:

```
Name [displayName] Required,User name [userPrincipalName] Required,Initial password [passwordProfile] Required,Block sign in (Yes/No) [accountEnabled] Required,First name [givenName],Last name [surname],Job title [jobTitle],Department [department],Usage location [usageLocation]
Test User 01,testuser01@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User01,Test Account,QA,US
Test User 02,testuser02@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User02,Test Account,QA,US
Test User 03,testuser03@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User03,Test Account,QA,US
Test User 04,testuser04@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User04,Test Account,QA,US
Test User 05,testuser05@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User05,Test Account,QA,US
Test User 06,testuser06@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User06,Test Account,QA,US
Test User 07,testuser07@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User07,Test Account,QA,US
Test User 08,testuser08@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User08,Test Account,QA,US
Test User 09,testuser09@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User09,Test Account,QA,US
Test User 10,testuser10@yourdomain.onmicrosoft.com,Contoso@1234!,No,Test,User10,Test Account,QA,US
```

7. Save the file as CSV (not .xlsx).

**Step 3: Upload and monitor**

8. Back in the Bulk create panel in the Entra admin center, click **Browse** (or drag-and-drop) to upload your CSV file.

9. Click **Submit**. The portal validates the file. If there are errors, it shows which rows and columns have problems.

10. After submission, click **Bulk operation results** from the notification or navigate to **Identity** → **Users** → **Bulk operation results**. The job appears with a status of **In progress** changing to **Completed**.

11. When the job completes, navigate to **All users**. Search for **Test User** — all 10 should now appear. The creation took one upload instead of 10 separate portal operations.

---

## Lab 2.6.2 — Bulk Update Users via PowerShell

**Tools:** PowerShell (Microsoft Graph PowerShell SDK)
**Prerequisite:** Lab 2.6.1 complete. The 10 test users exist in the tenant.

You will update all 10 test users to add a manager and change their department to `Testing` using a PowerShell loop.

1. Open PowerShell and connect to Microsoft Graph:

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

Sign in when prompted using your Global Administrator account.

2. Get the Object ID of Jordan Kim, who will be set as manager for the test users:

```powershell
$manager = Get-MgUser -Filter "displayName eq 'Jordan Kim'" -Property "Id,DisplayName"
$manager.Id
```

Note the Object ID in the output. This is Jordan Kim's unique identifier in the directory.

3. Get all test users by filtering on department:

```powershell
$testUsers = Get-MgUser -Filter "department eq 'QA'" -Property "Id,DisplayName,Department"
$testUsers | Select-Object DisplayName, Department
```

Expected output (10 rows):
```
DisplayName   Department
-----------   ----------
Test User 01  QA
Test User 02  QA
...
Test User 10  QA
```

4. Update all test users in a loop — change department to `Testing` and set Jordan Kim as manager:

```powershell
$managerRef = @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/users/$($manager.Id)"
}

foreach ($user in $testUsers) {
    # Update department
    Update-MgUser -UserId $user.Id -Department "Testing"

    # Set manager
    Set-MgUserManagerByRef -UserId $user.Id -BodyParameter $managerRef

    Write-Host "Updated: $($user.DisplayName)"
}
```

Expected output as the loop runs:
```
Updated: Test User 01
Updated: Test User 02
...
Updated: Test User 10
```

5. Verify the update on one user:

```powershell
Get-MgUser -UserId "testuser01@yourdomain.onmicrosoft.com" -Property "DisplayName,Department" | Select-Object DisplayName, Department
```

Expected output:
```
DisplayName   Department
-----------   ----------
Test User 01  Testing
```

6. Verify the manager was set:

```powershell
Get-MgUserManager -UserId "testuser01@yourdomain.onmicrosoft.com" | Select-Object -ExpandProperty AdditionalProperties | Select-Object displayName
```

Expected output:
```
displayName
-----------
Jordan Kim
```

---

## Lab 2.6.3 — Disable, Delete, and Restore Users

**Tools:** PowerShell, Entra admin center
**Prerequisite:** Lab 2.6.2 complete. All 10 test users exist.

**Step 1: Disable all test users**

1. In PowerShell, disable all 10 test users with a single command:

```powershell
$testUsers = Get-MgUser -Filter "department eq 'Testing'" -Property "Id,DisplayName"

foreach ($user in $testUsers) {
    Update-MgUser -UserId $user.Id -AccountEnabled $false
    Write-Host "Disabled: $($user.DisplayName)"
}
```

2. Verify one user is disabled:

```powershell
Get-MgUser -UserId "testuser01@yourdomain.onmicrosoft.com" -Property "DisplayName,AccountEnabled" | Select-Object DisplayName, AccountEnabled
```

Expected output:
```
DisplayName   AccountEnabled
-----------   --------------
Test User 01           False
```

**Step 2: Delete all test users via PowerShell**

3. Delete all test users:

```powershell
foreach ($user in $testUsers) {
    Remove-MgUser -UserId $user.Id
    Write-Host "Deleted: $($user.DisplayName)"
}
```

4. Navigate to the Entra admin center → **Identity** → **Users** → **All users**. The 10 test users should no longer appear.

5. Navigate to **Identity** → **Users** → **Deleted users**. All 10 test users should appear here. Note the **Deletion date** column — this is when the permanent deletion clock started.

**Step 3: Restore two users via the portal**

6. In the **Deleted users** list, click the checkbox next to **Test User 01**.

7. Click **Restore** at the top. Confirm when prompted.

8. Navigate back to **All users** and search for **Test User 01**. The account is restored and active.

9. Click on **Test User 01** → **Licenses**. Confirm that no licenses are assigned — licenses are not restored automatically. This user would need to be re-added to the Contoso-Licensed group to regain access.

**Step 4: Permanently delete remaining test users via PowerShell**

10. To clean up the remaining 9 deleted users permanently (without waiting 30 days), run:

```powershell
$deletedUsers = Get-MgDirectoryDeletedItemAsUser -Filter "department eq 'Testing'"

foreach ($user in $deletedUsers) {
    Remove-MgDirectoryDeletedItem -DirectoryObjectId $user.Id
    Write-Host "Permanently deleted: $($user.DisplayName)"
}
```

11. Also permanently delete Test User 01 (which was restored — disable and delete it first):

```powershell
$user01 = Get-MgUser -Filter "displayName eq 'Test User 01'" -Property "Id"
Update-MgUser -UserId $user01.Id -AccountEnabled $false
Remove-MgUser -UserId $user01.Id
```

Then permanently delete from the recycle bin:
```powershell
$deleted01 = Get-MgDirectoryDeletedItemAsUser -Filter "displayName eq 'Test User 01'"
Remove-MgDirectoryDeletedItem -DirectoryObjectId $deleted01.Id
```

12. Navigate to **Deleted users** in the portal. It should now be empty.

---

## Scenario Check

Contoso completed a bulk user creation exercise — 10 test accounts were created in a single CSV upload rather than 10 individual portal operations. A PowerShell script then updated all 10 with a new department value and manager assignment in a loop, demonstrating how attribute updates at scale work without touching the portal.

The disable-delete-restore sequence confirmed the 30-day soft-delete behavior. Test User 01 was restored after deletion — but without its license, requiring a group re-add to regain access. This is the exact sequence an admin would follow when a departing employee's account was deleted prematurely and needs to be recovered.

The 5 core Contoso users (Alex, Morgan, Jordan, Sam, River) were not affected by this module's operations — all test operations used the separate QA/Testing department users.

---

## Check Your Understanding

- [ ] Without referencing the module: name the four required columns in the bulk create CSV template
- [ ] From memory: what PowerShell command disables a user account using Microsoft Graph SDK?
- [ ] Navigate to **Identity** → **Users** → **Deleted users** — confirm it is empty after Lab 2.6.3
- [ ] Explain what happens to a user's license when they are deleted, and what an admin must do after restoring the user
- [ ] State how long Entra ID preserves a deleted user before permanent deletion
- [ ] Write the PowerShell command to get all users where department equals `Testing` — do it from memory

---

## Common Mistakes

**Building a CSV manually instead of downloading the template.**
The column headers must match exactly — including the text in square brackets and the word "Required." If you create your own CSV without the template, the upload will fail with a validation error that doesn't explain what's wrong. Always use the template from the portal.

**Deleting without checking for active sessions.**
When you disable or delete a user, their existing access tokens remain valid for up to one hour (until they expire naturally). If a user is a security concern, disabling the account and revoking their sessions separately is necessary. Module 6 covers session revocation.

**Assuming restore also restores access.**
It does not. The account comes back. The license assignment, group memberships, and app role assignments that were revoked during the deletion period must be manually restored. In a workflow-driven offboarding scenario, keeping a record of what was removed is essential.

**Running `Remove-MgUser` on a filter that's too broad.**
A typo in a `-Filter` parameter has deleted entire departments. Always test your filter with `Get-MgUser` first — see exactly which accounts match before running `Remove-MgUser`. Never run a deletion loop without first running the corresponding `Get-Mg*` command and verifying the output.

---

## What's Next

Module 2.7 covers B2B collaboration — guest users and external identity. Sam Taylor, who was created as an internal member account in Module 2.1, will be converted to the correct configuration: a proper B2B guest invite from an external identity. You will send the invitation, walk through the full redemption flow, and compare Sam's guest account object against Alex Lee's member account.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.5 — Licenses](../2-5-licenses-assigning-the-right-features/2-5-licenses-assigning-the-right-features.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.7 — Guest Users and B2B Collaboration →](../2-7-guest-users-and-b2b-collaboration/2-7-guest-users-and-b2b-collaboration.md)
