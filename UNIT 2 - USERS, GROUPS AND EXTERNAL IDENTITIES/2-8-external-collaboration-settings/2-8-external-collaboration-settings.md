# External Collaboration Settings and Cross-Tenant Access
*The defaults let anyone in your organization invite anyone from outside. That is not always what you want.*

> 🟢 **Beginner** · Unit 2, Module 2.8

---

## What You Will Learn

- Read and understand the External Collaboration Settings panel and what each setting controls
- Restrict guest invite permissions to admin-only
- Add a domain to the allowlist and understand the difference between allowlist and blocklist behavior
- Open Cross-Tenant Access Settings and understand the difference between default and organization-specific settings
- Configure inbound trust settings for a partner organization

---

## Why This Matters

*SC-300: Implement and manage user identities — Implement and manage external collaboration settings*

The default configuration of a new Entra ID tenant allows any internal member user to invite external guests. In a 50-person firm like Contoso, that means any employee can invite anyone from any external organization to access shared resources. Most organizations want more control than that. External Collaboration Settings and Cross-Tenant Access Settings (XTAP) are the two control surfaces for governing how external identities interact with your tenant. SC-300 tests both.

---

## 🔧 External Collaboration Settings — What Each Control Does

External Collaboration Settings is a single panel at **Identity → External Identities → External collaboration settings**. It controls four things:

**1. Guest invite settings**
Who in your organization can send B2B invitations.

| Option | Who can invite |
|---|---|
| Anyone in the organization can invite guest users including guests and non-admins (most permissive) | All members and all existing guests |
| Member users and users assigned to specific admin roles can invite guest users | Members only (no guests inviting guests) |
| Only users assigned to specific admin roles can invite guest users | Only Global Administrators, Guest Inviters, and User Administrators |
| No one in the organization can invite guest users including admins (most restrictive) | Nobody |

The default is the most permissive option. Most organizations should move to the second or third option.

**2. Guest user access restrictions**
What guests can see inside your directory after they accept an invitation.

| Option | What guests can see |
|---|---|
| Guest users have the same access as members (most permissive) | Everything a member can see — all users, groups, directory objects |
| Guest users have limited access to properties and memberships of directory objects | Can see their own profile; cannot enumerate all users or groups |
| Guest users have the most restricted access to directory objects (most restrictive) | Cannot look up any other user or group in the directory |

The default is the middle option. The most restrictive option is appropriate when guests should not be able to see who else is in the organization.

**3. Allow/deny specific external domains (guest invite restrictions)**
A domain allowlist or blocklist for incoming B2B invitations.

- **Allowlist:** Invitations can only be sent to users at the domains listed. Inviting anyone outside those domains is blocked.
- **Blocklist:** Invitations can be sent to anyone except users at the domains listed.
- **Neither (default):** Invitations can be sent to any external domain.

You cannot use both allowlist and blocklist simultaneously — choosing one disables the other.

**4. Enable guest self-service sign-up via user flows**
Allows external users to sign up for access without requiring an admin invitation. Covered in Module 8.

---

## 🏢 Cross-Tenant Access Settings — The Newer, More Granular Model

External Collaboration Settings governs whether you can invite guests from specific domains. Cross-Tenant Access Settings (XTAP) governs what happens when someone from another Microsoft Entra tenant tries to access your resources — or when your users try to access another organization's resources.

XTAP has two scopes:

**Default settings:** Apply to all external Entra tenants that do not have an organization-specific setting. This is the starting point.

**Organization-specific settings:** Configured for a specific partner tenant by their tenant ID or domain. These override the default settings for that organization only.

Each scope has two directions:

| Direction | What it controls |
|---|---|
| **Inbound** | How you trust users coming from an external tenant into your tenant |
| **Outbound** | How you allow your users to access resources in an external tenant |

And within each direction, two things can be configured:

| Setting | What it does |
|---|---|
| **B2B collaboration** | Whether B2B guest invitations are allowed to/from this tenant |
| **B2B direct connect** | A deeper trust for Teams Connect shared channels (not covered in this module) |
| **Trust settings (inbound only)** | Whether you trust MFA, compliant devices, and hybrid Azure AD joined claims from the partner tenant |

> 🔑 **Key concept:** The trust settings are the most important part of XTAP for an admin. If you require MFA via Conditional Access and a guest user from a partner tenant has already completed MFA at their home organization, you can configure XTAP to trust that MFA claim — the guest is not prompted for MFA again. This prevents double-MFA friction in partner scenarios.

---

## Lab 2.8.1 — Review and Tighten External Collaboration Settings

**Tools:** Entra admin center
**Prerequisite:** None

1. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

2. In the left menu, navigate to **Identity** → **External Identities** → **External collaboration settings**.

3. The settings page opens. You will see four sections. Read through each section before making any changes.

**Review Guest invite settings:**

4. Under **Guest invite settings**, note the current selection. In a new developer tenant, the default is usually **Anyone in the organization can invite guest users including guests and non-admins**.

5. Change the selection to **Only users assigned to specific admin roles can invite guest users including admins**. This means only Global Administrators, User Administrators, and users in the Guest Inviter role can send invitations.

6. Click **Save** at the top of the page. You should see a success notification.

**Review Guest user access restrictions:**

7. Under **Guest user access restrictions**, note the current selection. The default is typically **Guest users have limited access to properties and memberships of directory objects**.

8. Leave this setting at the default for now — this is appropriate for most environments. Make a note of what the most restrictive option says.

**Review collaboration restrictions:**

9. Under **Collaboration restrictions**, note the current selection. The default is **Allow invitations to any domain (most inclusive)**.

10. Change the setting to **Allow invitations only to the specified domains (most restrictive)**.

11. In the text box that appears, type: `microsoft.com`

12. Click **Save**. You have now restricted guest invitations to users with `@microsoft.com` email addresses only.

> ⚠️ **Warning:** This domain restriction now applies to new invitations. If you try to invite a user from a different domain, the invitation will be blocked. For the next lab, you will need to revert this setting — leave it in place for now to observe the effect, then revert it after Lab 2.8.2.

---

## Lab 2.8.2 — Test the Domain Restriction and Revert

**Tools:** Entra admin center
**Prerequisite:** Lab 2.8.1 complete — domain allowlist set to `microsoft.com`

1. Navigate to **Identity** → **Users** → **All users** → **Invite external user**.

2. Try to invite a user with an email address outside `microsoft.com` — use a Gmail address or any address that is not `@microsoft.com`.

3. Fill in the name and email. Click **Invite**.

4. The invitation is blocked. You will see an error: *"You can't invite [email] because invitations to external domains are not allowed for your tenant configuration"* or similar. The exact wording may vary.

5. This confirms the domain restriction is working.

**Revert the settings:**

6. Navigate back to **Identity** → **External Identities** → **External collaboration settings**.

7. Under **Collaboration restrictions**, change back to **Allow invitations to any domain (most inclusive)**.

8. Under **Guest invite settings**, change back to **Member users and users assigned to specific admin roles can invite guest users** — this is the recommended middle ground for most organizations (members can invite, but guests cannot invite other guests).

9. Click **Save**.

---

## Lab 2.8.3 — Explore Cross-Tenant Access Settings

**Tools:** Entra admin center
**Prerequisite:** None

1. Navigate to **Identity** → **External Identities** → **Cross-tenant access settings**.

2. The Cross-tenant access settings page opens. You will see two tabs: **Default settings** and **Organizational settings**.

**Review Default settings:**

3. Click the **Default settings** tab if it's not already selected.

4. You will see the inbound and outbound default configuration. The page shows two sections:
   - **Inbound access settings** — how external tenants' users can access your tenant
   - **Outbound access settings** — how your users can access external tenants

5. Under **Inbound access settings**, click **Edit inbound defaults**.

6. The inbound settings panel opens with three tabs:
   - **B2B collaboration** — whether B2B guest invites from all external tenants are allowed
   - **B2B direct connect** — Teams Connect shared channels (leave default)
   - **Trust settings** — whether you trust MFA and device compliance claims from external tenants

7. Select the **Trust settings** tab. Review the three options:
   - **Trust multifactor authentication from Microsoft Entra tenants** — if enabled, a guest who completed MFA at their home tenant satisfies your MFA Conditional Access policy
   - **Trust compliant devices** — if enabled, a device marked compliant at the home tenant is trusted as compliant in your tenant
   - **Trust hybrid Azure AD joined devices** — if enabled, a device hybrid-joined at the home tenant is trusted

8. Leave all three trust settings at their defaults (typically all unchecked for new tenants). Click **Cancel** to close without changing.

**Add an organization-specific setting:**

9. Click the **Organizational settings** tab.

10. Click **Add an organization**.

11. A panel opens asking for a tenant domain or tenant ID. Type `microsoft.com` in the field. Click **Add**.

12. A new row appears for `microsoft.com` in the organizational settings table. This is a placeholder — no specific settings are configured yet, meaning it will use the defaults.

13. Click on the `microsoft.com` row to open the organization-specific settings panel.

14. Select the **Inbound access** tab → **Trust settings**.

15. Enable **Trust multifactor authentication from Microsoft Entra tenants** for this organization. This means: if a user from Microsoft's tenant has already completed MFA at Microsoft's Entra ID, your tenant's Conditional Access MFA requirement will treat them as already satisfying MFA — no second MFA prompt.

16. Click **Save**. The microsoft.com organization now has a trust setting that overrides the default for inbound users from that specific tenant.

17. Click the **X** on the panel to close. The Organizational settings tab now shows `microsoft.com` with a **Customized** indicator.

18. Delete this organizational setting — it was for lab purposes only. Click on the `microsoft.com` row, then click **Delete** to remove the organization-specific setting.

---

## Scenario Check

Contoso's external collaboration posture is now configured. Guest invite permissions are set to allow member users to invite guests (but not allow guests to invite other guests) — this is the appropriate middle ground for a 50-person professional services firm. The domain restriction test confirmed that when the allowlist is active, invitations outside the allowed domains are blocked at submission time.

The Cross-Tenant Access Settings exploration revealed the trust model: by default, Contoso does not trust MFA or device compliance claims from any external tenant. If Contoso partners with another Microsoft 365 organization in the future, an organization-specific XTAP setting can be added to trust that partner's MFA, reducing friction for partner users accessing shared resources.

Sam Taylor's guest account was deleted in Module 2.7. The invitation settings mean that re-inviting Sam (when needed) would go through Jordan Kim or another admin — not through a general employee.

---

## Check Your Understanding

- [ ] Navigate to **External collaboration settings** — confirm the Guest invite setting is set to members only, not the most permissive option
- [ ] From memory: state the difference between the allowlist and blocklist options in Collaboration restrictions
- [ ] Navigate to **Cross-tenant access settings** → **Organizational settings** — confirm the microsoft.com entry was deleted
- [ ] Explain in one sentence what "Trust multifactor authentication from Microsoft Entra tenants" means in the XTAP trust settings
- [ ] State why allowing guests to invite other guests is a security risk worth restricting
- [ ] Name the two scopes in Cross-Tenant Access Settings (not inbound/outbound — the two tabs)

---

## Common Mistakes

**Setting the allowlist to one domain and forgetting to revert.**
If you configure an allowlist and never revert it, every new guest invitation attempt from outside that domain will fail silently for the admin trying to send it. The error is not obvious. Always document any changes to External Collaboration Settings and track them in your change management process.

**Confusing External Collaboration Settings with Cross-Tenant Access Settings.**
External Collaboration Settings controls who in your organization can send invitations and which domains are allowed. Cross-Tenant Access Settings controls the trust relationship between Entra tenants — including whether MFA and device compliance from another tenant are accepted. They are different panels with different purposes.

**Enabling XTAP trust settings without a Conditional Access policy to enforce them.**
Trusting MFA from a partner tenant only matters if you have a Conditional Access policy requiring MFA. If you have no CA policy requiring MFA, the trust setting does nothing — there is nothing to trust against. Configure XTAP trust settings alongside the CA policies they support.

**Leaving organization-specific XTAP settings after a project ends.**
A project with a vendor ends. The vendor's tenant still has a customized XTAP setting in your organizational settings list. Former vendor employees may still be able to access resources based on that trust configuration. Review XTAP organizational settings as part of vendor offboarding.

---

## What's Next

Module 2.9 covers Administrative Units — a way to delegate admin tasks to specific groups without giving full tenant-wide admin rights. Jordan Kim manages IT at Contoso, but Jordan does not need Global Administrator — they only need to manage IT department users. Administrative Units enable that scope restriction.

---

> 📚 **Entra ID — Zero to Admin** · Unit 2: Users, Groups and External Identities
>
> [← Module 2.7 — Guest Users and B2B Collaboration](../2-7-guest-users-and-b2b-collaboration/2-7-guest-users-and-b2b-collaboration.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 2.9 — Administrative Units →](../2-9-administrative-units/2-9-administrative-units.md)
