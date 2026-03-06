# Guest Users and B2B Collaboration
*An external user in your tenant is not the same as an internal user. The difference matters for access, licensing, and lifecycle.*

> 🟢 **Beginner** · Unit 2, Module 2.7

---

## What You Will Learn

- Explain how B2B collaboration works and what a guest account object looks like in Entra ID
- Invite Sam Taylor as a proper B2B guest and walk through the full redemption flow
- Compare a guest account object against a member account — every field that differs
- Assign a guest user access to a specific resource and verify they can only access what was assigned
- Manage the guest account lifecycle: resend an invite, remove access, and soft-delete the guest

---

## Why This Matters

*SC-300: Implement and manage user identities — Implement and manage external identities including B2B collaboration*

Most organizations share resources with contractors, vendors, and partners. B2B collaboration is the secure way to do it — the external user signs in with their own identity (Microsoft account, work account, or Google account) and gets access to specific resources in your tenant. You do not create a password for them. You do not manage their authentication. They authenticate with their home organization or personal identity provider. Your tenant only decides what they can access.

In Module 2.1, Sam Taylor was created as a member account in your tenant — with a password you control, a UPN in your domain, and no external identity relationship. That was necessary to demonstrate member user creation. The correct configuration for a contractor is a B2B guest. This module fixes that.

---

## 🔑 What B2B Collaboration Actually Is

B2B stands for Business-to-Business. In Entra ID, B2B collaboration means: an identity that lives outside your tenant gets represented inside your tenant as a guest account object.

The guest account is a shadow object. It stores the external user's display name, email address, and a reference to their home tenant or identity provider. It does not store their password. It does not store their credentials. When Sam Taylor signs in to access your tenant's resources, the authentication happens against Sam's home organization (or their personal Microsoft or Google account) — not against your tenant. Your tenant only evaluates what Sam is allowed to access after authentication succeeds.

This distinction is important for two reasons:
1. **Password management is not your problem.** If Sam's password is compromised, Sam's IT team handles it.
2. **You cannot see Sam's home tenant data.** The guest relationship is one-directional.

---

## 👤 Guest Account Object — What Differs from a Member

When a guest accepts an invitation, a user object is created in your tenant with specific attributes that distinguish it from member accounts:

| Property | Member account | Guest account |
|---|---|---|
| **User type** | `Member` | `Guest` |
| **UPN format** | `sam@yourdomain.onmicrosoft.com` | `sam_externaldomain.com#EXT#@yourdomain.onmicrosoft.com` |
| **Password managed by** | Your tenant | External identity provider |
| **Mail attribute** | Set to your tenant domain | Set to their external email address |
| **Creation type** | `LocalAccount` | `Invitation` |
| **External user state** | N/A | `PendingAcceptance` → `Accepted` |
| **Default directory access** | Can see all users and groups | Restricted by default |

The UPN for a guest contains `#EXT#` — this is how Entra ID signals that the account is a guest. The original email address is embedded before `#EXT#`. You cannot change this UPN format.

> 🔑 **Key concept:** A guest account does not consume a Microsoft 365 license from your tenant. Guests have free access to participate in Teams, read SharePoint sites they're shared on, and use Entra ID features up to the B2B free tier allowance. The guest user's license — if any — comes from their home organization.

---

## 📨 Three Ways to Invite a Guest

| Method | How | Best for |
|---|---|---|
| **Admin invite via portal** | Identity → Users → Invite external user | One-off invites for known individuals |
| **Direct sharing link** | Share a specific SharePoint site or Teams channel — Entra ID generates the invite | Resource owners inviting collaborators directly |
| **Self-service B2B signup** | Allow external users to request access via My Access portal | Large-scale external collaboration programs |

This module uses the admin invite via portal — the method you will use most often for contractor and vendor access.

---

## Lab 2.7.1 — Delete the Member Account and Invite Sam Taylor as a Guest

**Tools:** Entra admin center
**Prerequisite:** Sam Taylor exists as a member account from Module 2.1. You need access to a second email address to receive the guest invitation (use a personal Microsoft account or a Gmail account — this represents Sam's external identity).

**Step 1: Delete Sam Taylor's member account**

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. Navigate to **Identity** → **Users** → **All users**.

3. Find and click **Sam Taylor**.

4. At the top of the profile page, click **Delete**. Confirm the deletion.

5. Sam Taylor's member account is now in the deleted users recycle bin. Do not restore it — the guest account will replace it.

**Step 2: Invite Sam Taylor as a B2B guest**

6. Navigate to **Identity** → **Users** → **All users**.

7. Click **Invite external user** at the top (not "New user" — that creates a member). The invite external user panel opens.

8. Fill in the invitation details:
   - **Name:** `Sam Taylor`
   - **Email address:** Enter the external email address you are using to represent Sam (your personal Microsoft account, Gmail, or any valid external email)
   - **First name:** `Sam`
   - **Last name:** `Taylor`
   - **Personal message:** Type a brief message — this appears in the invitation email Sam receives

9. Scroll down to the **Properties** section. Set:
   - **Job title:** `Contractor`
   - **Department:** `Contractors`
   - **Usage location:** `United States`

10. Click **Invite**. The invitation is sent. The portal shows a success notification.

**Step 3: Accept the invitation as Sam**

11. Open the email inbox of the external email address you used for Sam's invitation.

12. Find the email with subject line: **[Your organization] invited you to access applications within their organization** (or similar phrasing). The sender will be `invites@microsoft.com`.

13. Click **Accept invitation** in the email.

14. A browser window opens asking you to sign in with the invited email address. Sign in.

15. You will see a permissions review screen: the page shows what your tenant will be able to see about Sam's account. Review and click **Accept**.

16. Sam is now redirected to the My Apps portal (`myapps.microsoft.com`) showing the applications they have access to. Currently, Sam has no apps assigned — this is correct.

**Step 4: Verify the guest account in the portal**

17. Back in the Entra admin center, navigate to **Identity** → **Users** → **All users**.

18. Search for **Sam Taylor**. The account appears. Note the **User type** column shows **Guest** (member accounts show **Member**).

19. Click on **Sam Taylor** to open the profile. Observe:
   - **User principal name:** Contains `#EXT#`
   - **User type:** Guest
   - **External user state:** Accepted
   - **Source:** Microsoft Account (or Google account, depending on what you used)

---

## Lab 2.7.2 — Compare Sam's Guest Account to Alex Lee's Member Account

**Tools:** Entra admin center
**Prerequisite:** Lab 2.7.1 complete

Open both accounts in separate browser tabs and work through the comparison.

1. Navigate to **Identity** → **Users** → **All users** → **Sam Taylor** → **Properties**. Keep this tab open.

2. Open a new tab and navigate to **Identity** → **Users** → **All users** → **Alex Lee** → **Properties**.

For each property below, compare what you see in Sam's profile vs Alex's profile:

| Property | Sam Taylor (Guest) | Alex Lee (Member) |
|---|---|---|
| User type | Should show `Guest` | Should show `Member` |
| User principal name | Contains `#EXT#` | Standard `@yourdomain.onmicrosoft.com` |
| Source | Microsoft Account or Google | Azure Active Directory (or just your tenant name) |
| External user state | `Accepted` | Not shown — internal users don't have this field |
| Password managed by | External (greyed out, cannot reset from your portal) | Your tenant (you can reset it) |

3. In Sam's profile, click **Reset password**. You will see a message: *"You can't reset the password for this user because Sam Taylor uses an external account for sign-in."* This confirms the credential boundary — you do not own Sam's credentials.

4. In Alex's profile, click **Reset password**. The reset option works normally — a temporary password can be generated.

---

## Lab 2.7.3 — Assign Sam Access to a Specific Resource

**Tools:** Entra admin center
**Prerequisite:** Engineering-Team SharePoint site exists from Module 2.3

1. Navigate to **Identity** → **Groups** → **All groups** → **Finance-All** (the Security Group from Module 2.3).

2. Select **Members** → **Add members**. Search for **Sam Taylor** and add them. Click **Save**.

3. Sam Taylor is now a member of Finance-All. If Finance-All has been granted access to a SharePoint site (from Module 2.3), Sam now has that access as well.

4. Open the SharePoint site that Finance-All was granted access to. Navigate there using the site URL or through the SharePoint admin center.

5. In the site settings, go to **Settings** → **Permissions** (or **Site permissions** depending on your SharePoint version). Verify that Finance-All is listed with the permissions you assigned in Module 2.3, and that this grants Sam Taylor access through group membership.

**Verify Sam can access the site:**

6. Open an InPrivate or private browsing window.

7. Navigate to `myapps.microsoft.com` and sign in as Sam Taylor using the external email address.

8. From My Apps, open the SharePoint site (if it appears as a pinned app) or navigate to the site URL directly.

9. Sam should have access to the site with the permissions granted through Finance-All. Sam should not have access to other sites or resources that were not explicitly shared.

---

## Lab 2.7.4 — Remove Sam's Access and Manage Guest Lifecycle

**Tools:** Entra admin center
**Prerequisite:** Lab 2.7.1 complete

**Remove Sam from Finance-All:**

1. Navigate to **Identity** → **Groups** → **All groups** → **Finance-All** → **Members**.

2. Select **Sam Taylor** and click **Remove member**. Confirm.

3. Sam loses the resource access that Finance-All granted. They still have a guest account object in your tenant, but no access to any resources.

**Soft-delete the guest account:**

4. Navigate to **Identity** → **Users** → **All users** → **Sam Taylor**.

5. Click **Delete**. Confirm.

6. Sam's guest account moves to **Deleted users**. The external user's home account is not affected — only your tenant's guest object is removed.

7. Navigate to **Identity** → **Users** → **Deleted users**. Sam Taylor should appear.

8. Do not restore Sam yet — a future module will re-invite Sam for B2B governance scenarios.

> 💡 **Tip:** Deleting a guest account does not notify the external user. They will simply find that the invitation link and any resources they had access to no longer work. If you want to communicate the removal, do so separately before deleting the account.

---

## Scenario Check

Sam Taylor now exists as a proper B2B guest — authenticated by their external identity provider, with a guest account object in the Contoso tenant that contains `#EXT#` in the UPN. The original member account from Module 2.1 has been deleted and replaced.

The comparison lab confirmed the key difference: Sam's password cannot be reset from the Contoso admin center. Contoso does not own Sam's credentials. When Sam signs in to access Contoso resources, the authentication challenge goes to Sam's home identity provider — not to Contoso's Entra ID.

Sam's access to Finance-All was granted and then removed within this module. The guest account object was then deleted, placing it in the 30-day recycle bin. The external user's actual account at their home organization was unaffected — only the Contoso shadow object was removed.

---

## Check Your Understanding

- [ ] Open **Deleted users** and verify Sam Taylor's guest account is there, not in All users
- [ ] From memory: state the UPN format for a guest account (describe what #EXT# means and where it appears)
- [ ] Name two reasons why Contoso cannot reset a guest user's password from the Entra admin center
- [ ] Explain the difference between deleting a guest account in your tenant vs deleting the guest's home account
- [ ] State what happens to Sam's home organization identity when you delete Sam's guest account in Contoso
- [ ] Without referencing the module: name three properties that differ between a guest account and a member account

---

## Common Mistakes

**Creating a member account for an external user instead of sending a B2B invite.**
This is exactly what Module 2.1 did for demonstration purposes. In production, it means you control the credentials for someone outside your organization, you must manage their password, and their account has no connection to their real work identity. Always use B2B for external users. The only exception: a service account with no human behind it.

**Assuming a guest account costs a Microsoft 365 license.**
It does not — not for basic collaboration. A guest user who reads SharePoint, participates in Teams, and uses basic app access does not consume a seat from your tenant. They consume a license from their home tenant (if they have one). The licensing exception: if you assign a paid Entra ID feature (like PIM or access packages) to a guest, that specific feature requires a license in your tenant.

**Leaving guest accounts in the tenant indefinitely.**
Guest accounts for completed projects should be removed. A former contractor who still has a guest account and remembers the direct URL to a SharePoint site can still access it. Guest lifecycle management — access reviews for guests — is covered in Module 8.

**Inviting a guest to a group with broad access without checking what the group grants.**
Adding Sam to Finance-All gives Sam everything Finance-All can reach — every SharePoint site, every app role, every CA exclusion. Review group permissions before adding a guest member.

---

## What's Next

Module 2.8 covers External Collaboration Settings and Cross-Tenant Access — the controls that govern who in your organization can invite guests, which external domains are allowed, and how to set up trust relationships with specific partner organizations. The guest invitation Sam received was possible because the default external collaboration settings allow it. Module 2.8 shows what those defaults are and how to restrict or expand them.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.6 — Bulk Operations](../2-6-bulk-operations/2-6-bulk-operations.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.8 — External Collaboration Settings →](../2-8-external-collaboration-settings/2-8-external-collaboration-settings.md)
