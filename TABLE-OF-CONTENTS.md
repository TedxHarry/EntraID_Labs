# Entra ID — Zero to Admin
### A Hands-On Learning Series

> **Who this is for:** Complete beginners. No prior experience with Entra ID, Azure, or identity management required.
>
> **What you'll have at the end:** The skills to work as an Entra ID administrator — and the knowledge to pass the SC-300 certification exam.
>
> **How it works:** Every module has a concept section and a hands-on lab. You don't just read — you do. Every lab uses your free Microsoft 365 E5 developer tenant. You break things, fix them, and learn by doing.

---

## Before You Start

**Set up your free practice environment first.**
Every lab in this series uses a free Microsoft 365 E5 developer tenant.
Go to `developer.microsoft.com/microsoft-365/dev-program` and create your free sandbox before starting Module 1.1.

**Tools you'll use throughout this series:**

| Tool | What it's for | URL |
|---|---|---|
| Entra Admin Center | Your primary admin workspace | `entra.microsoft.com` |
| Microsoft 365 Admin Center | User and license management | `admin.microsoft.com` |
| Graph Explorer | Test Microsoft Graph API calls in your browser | `aka.ms/ge` |
| Microsoft Graph PowerShell | Automate admin tasks via PowerShell | Install: `Install-Module Microsoft.Graph` |
| jwt.ms | Decode and inspect access tokens | `jwt.ms` |
| My Access Portal | See what access looks like for end users | `myaccess.microsoft.com` |
| My Sign-Ins | See sign-in history as a user | `mysignins.microsoft.com` |

---

## The Running Scenario

Throughout this series you'll build a complete identity environment for a fictional company: **Contoso Ltd** — a 50-person professional services firm moving from legacy IT to Microsoft 365.

As you progress through the units, you'll hire employees, secure their access, onboard contractors, connect SaaS apps, respond to security incidents, and eventually hand over a production-ready Entra ID configuration.

**Your characters:**

| Name | Role | Department | What they're for |
|---|---|---|---|
| Alex Lee | Finance Analyst | Finance | Your primary test user — beginner labs and most policy targets |
| Morgan Chen | Software Engineer | Engineering | Group and app access labs |
| Jordan Kim | IT Administrator | IT | PIM, privileged access, and approval workflow labs |
| Sam Taylor | Contractor | External | Guest identity, B2B collaboration, and governance labs |
| River Patel | HR Manager | HR | Lifecycle workflows, access approvals, and manager-driven labs |

You'll create these identities in Unit 2 and use them throughout the series.

---

## Series Overview

| Unit | Theme | Modules | What you'll be able to do |
|---|---|---|---|
| 1 | Getting Started | 4 | Navigate Entra ID, understand what it is and why it exists |
| 2 | Users, Groups and External Identities | 9 | Create, manage, and organize identities at scale — including guests and delegated admin |
| 3 | Authentication | 7 | Configure MFA, passwordless, SSPR, authentication policies, and session controls |
| 4 | Conditional Access | 8 | Build and manage a complete CA policy stack, from Security Defaults through enforcement |
| 5 | Applications and SSO | 9 | Connect SaaS apps, configure SSO, manage permissions and consent |
| 6 | Identity Security | 6 | Detect and respond to account compromise and risky sign-ins |
| 7 | Privileged Access | 6 | Configure PIM, JIT access, and role governance |
| 8 | Identity Governance | 5 | Run access reviews, entitlement management, lifecycle automation |
| 9 | Devices and Compliance | 7 | Enroll devices, enforce compliance, integrate with Intune |
| 10 | Hybrid Identity | 7 | Connect on-premises AD to Entra ID |
| 11 | Workload Identities | 6 | Secure apps and automation without passwords |
| 12 | Monitoring and Operations | 6 | Read logs, set up alerts, build runbooks |
| 13 | Architecture and Expert Design | 6 | Design systems, prepare for SC-300, become job-ready |

**Total: 86 modules**

---

---

## UNIT 1 — Getting Started
*Understand what Entra ID is, how it works, and how to navigate your workspace before touching any settings.*

> **SC-300 mapping:** Foundational knowledge for all exam domains
> **Estimated time:** 3–4 hours

---

### 1.1 — What Is Microsoft Entra ID?
**You will learn:** The problem Entra ID solves, the difference between Entra ID and on-premises Active Directory, the three things Entra ID manages (identities, resources, policies).

**Lab:** Create your free developer tenant. Tour the Entra admin center. Create your first test user. Find your first sign-in in the logs.

**You'll be able to:** Explain what Entra ID is to someone who has never heard of it. Create a basic user account. Navigate to the five most-used admin sections.

---

### 1.2 — Understanding Your Tenant
**You will learn:** What a tenant is, what lives inside it, what your Tenant ID and primary domain are, why tenant design decisions made early are hard to undo later.

**Lab:** Find your Tenant ID and primary domain. Explore tenant-level settings. Add a custom domain name (optional). Understand the difference between the `.onmicrosoft.com` domain and a custom domain.

**You'll be able to:** Describe what a tenant is and what it contains. Find any tenant's ID. Explain why you can't merge tenants.

---

### 1.3 — The Entra Admin Center — Your Workspace
**You will learn:** How the admin center is organized, where every major feature lives, how to use search to navigate, the difference between the Entra admin center and the Microsoft 365 admin center.

**Lab:** Navigate to every major section covered in this series — find each one using the left nav and using search. Pin your top five most-used sections. Set up your admin center bookmarks.

**You'll be able to:** Navigate to any Entra ID feature in under 30 seconds without searching.

---

### 1.4 — How Entra ID Makes Decisions
**You will learn:** The complete sign-in flow from first keystroke to accessing an app — what happens at each step, what can fail at each step, and what logs capture each step. This is the mental model everything else in the series builds on.

**Lab:** Sign in as a test user. Find that sign-in in the logs. Read every field in the log entry. Decode an access token at jwt.ms. Identify the exact step where a simulated failure occurred.

**You'll be able to:** Trace any sign-in from start to finish. Read a sign-in log entry and explain exactly what happened and why.

---

---

## UNIT 2 — Users, Groups and External Identities
*Learn to create, organize, and manage identities — internal employees, external guests, and delegated administration.*

> **SC-300 mapping:** Domain 1 — Implement and manage user identities
> **Estimated time:** 8–9 hours

---

### 2.1 — Creating and Managing Users
**You will learn:** How to create users in the portal and with PowerShell, the difference between cloud-only and synced users, what each user property means and when it matters, how the Object ID differs from the UPN.

**Lab:** Create all five Contoso characters (Alex, Morgan, Jordan, Sam, River) with the correct properties: department, job title, usage location. Do the first three in the portal, the last two using PowerShell. Verify each one in the sign-in logs after creation.

**You'll be able to:** Create users both in the portal and via PowerShell. Set all required properties. Explain why the Object ID is more important than the display name.

---

### 2.2 — User Properties and What They Drive
**You will learn:** Which user properties are just informational vs which ones actively drive other features (department drives dynamic groups, usage location is required for licensing, manager drives approval workflows).

**Lab:** Update the Contoso users with manager relationships (River manages Alex and Morgan). Change Alex's department and watch it propagate to a dynamic group in real time. See what breaks when usage location is missing.

**You'll be able to:** Set user properties that drive automation and policies. Understand which properties are cosmetic and which are functional.

---

### 2.3 — Security Groups vs Microsoft 365 Groups
**You will learn:** What security groups do (access control, policy targeting, license assignment), what Microsoft 365 Groups do (Teams workspaces, SharePoint sites, shared mailboxes), when to use each, and why mixing them up causes problems.

**Lab:** Create a Security Group for Finance users. Create an M365 Group for the Engineering team. Add the correct Contoso users to each. Assign a SharePoint site permission via group rather than directly to a user.

**You'll be able to:** Choose the right group type for any scenario. Explain to a colleague why they shouldn't use an M365 Group for license assignment.

---

### 2.4 — Dynamic Groups — Auto-Membership from Attributes
**You will learn:** How dynamic membership rules work, how to write membership rules using user attributes, how long membership changes take to propagate, and why dynamic groups are better than manually maintained groups at scale.

**Lab:** Create a dynamic group that automatically includes all users where `department = Finance`. Add a new Finance user — watch them join automatically. Change a user's department — watch them leave automatically. Write a rule that uses multiple conditions (department AND country).

**You'll be able to:** Write dynamic group rules for common real-world scenarios. Troubleshoot why a user isn't appearing in a dynamic group.

---

### 2.5 — Licenses — Assigning the Right Features
**You will learn:** How Microsoft 365 licensing works, why you can't use features you haven't licensed, how to assign licenses via group (correct way) vs per-user (bad at scale), and what happens when a license is removed.

**Lab:** Assign Microsoft 365 E5 licenses to all five Contoso users via group membership. Remove Alex from the license group — watch their apps disappear. Re-add them. Verify which Entra ID features require P1 vs P2.

**You'll be able to:** Assign and manage licenses at scale using groups. Know which features require which license tier without looking it up.

---

### 2.6 — Bulk Operations — Managing Users at Scale
**You will learn:** How to create multiple users via CSV upload, how to do bulk updates, how to use PowerShell to manage hundreds of users at once, and why individual portal operations don't scale.

**Lab:** Create 10 test users in one operation using a CSV upload. Update all 10 with department and usage location using a PowerShell script. Disable all 10 with a single PowerShell command. Delete them. Restore them from the deleted users recycle bin.

**You'll be able to:** Onboard a group of users at once. Run bulk updates via PowerShell. Recover accidentally deleted users.

---

### 2.7 — Guest Users and B2B Collaboration
**You will learn:** What B2B collaboration is and how it differs from creating internal users, how the guest invitation flow works (email invite vs direct link vs self-service), what a guest account object looks like in Entra ID compared to a member account, what guests can and cannot access in your tenant by default, and how to manage guest account lifecycle.

**Lab:** Invite Sam Taylor as a guest user to your tenant. Send the invitation email and go through the full redemption flow — accept the invite as Sam, watch the guest account appear in Entra ID. Open Sam's account object and compare every property to Alex Lee's member account. Assign Sam access to one SharePoint site and one Enterprise Application. Verify Sam can access only what was assigned. Resend an invite to an expired guest. Remove Sam's access and soft-delete the guest account.

**You'll be able to:** Invite external users via B2B. Manage guest account properties and access. Explain how a guest account differs from a member account.

---

### 2.8 — External Collaboration Settings and Cross-Tenant Access
**You will learn:** The External Collaboration Settings panel that controls who in your organization can invite guests, which external domains are allowed or blocked, and whether guests can see your directory. What Cross-Tenant Access Settings (XTAP) are — the newer, more granular way to control inbound and outbound trust between your tenant and specific partner organizations or all external tenants.

**Lab:** Open Identity → External Identities → External collaboration settings. Restrict guest invite permissions to "Only users assigned to specific admin roles can invite guest users." Add a domain to the allowlist. Open Cross-Tenant Access Settings. Find the Default settings — review the inbound and outbound defaults. Add an organization-specific setting: configure inbound trust for a partner tenant (use a secondary Microsoft account or your personal tenant as the "partner"). Verify the settings took effect.

**You'll be able to:** Configure who can invite guests and from which organizations. Set up cross-tenant access policies for partner relationships.

---

### 2.9 — Administrative Units — Delegated Administration at Scale
**You will learn:** What Administrative Units are and the problem they solve — in a large organization, a User Administrator can manage every user in the tenant, which violates least-privilege. AUs let you scope admin roles to a specific subset of users, groups, or devices. How to create AUs, populate them, and assign scoped roles within them.

**Lab:** Create an Administrative Unit called "Finance Division." Add Alex Lee and the Finance security group to the AU. Assign Jordan Kim the User Administrator role — but scope it only to the Finance AU. Sign in as Jordan Kim. Attempt to reset Alex Lee's password (succeeds — Alex is in the AU). Attempt to reset Morgan Chen's password (blocked — Morgan is outside the AU). Verify the scope enforcement in the audit log.

**You'll be able to:** Create Administrative Units, add users and groups to them, and assign admin roles scoped to an AU. Explain when AUs are the right solution for delegated administration.

---

---

## UNIT 3 — Authentication
*Control how people prove who they are — and make it secure without making it painful.*

> **SC-300 mapping:** Domain 2 — Implement authentication and access management
> **Estimated time:** 7–8 hours

---

### 3.1 — From Password to Token: What Authentication Actually Produces
**You will learn:** What Entra ID issues when a sign-in succeeds — the three token types (ID token, access token, refresh token) and what each one is for. What claims live inside a token (who the user is, what tenant they're from, what they're allowed to do, when the token expires). How apps use these tokens to make access decisions. This goes deeper than the high-level sign-in flow in Module 1.4 — it covers what's inside the authentication result.

**Lab:** Sign in as Alex Lee using a browser. Use the jwt.ms tool to decode an access token — identify the issuer (`iss`), subject (`sub`), audience (`aud`), object ID (`oid`), scope (`scp`), and expiry (`exp`) claims. Now update Alex's job title in the Entra admin center and decode a fresh token — find the updated claim. Use Graph Explorer to call the `/me` endpoint — see how the token claims translate into the profile data the app receives.

**You'll be able to:** Decode any Entra ID token and explain what each key claim means. Explain the difference between an ID token and an access token. Understand why apps trust tokens rather than calling Entra ID on every request.

---

### 3.2 — The Authentication Methods Policy
**You will learn:** The Authentication Methods Policy is the single place that controls which authentication methods your entire tenant can use. How it replaces legacy per-user MFA settings. What each method is and how strong it is.

**Lab:** Open the Authentication Methods Policy. Enable Microsoft Authenticator for all users. Enable FIDO2 Security Keys. Disable SMS for most users (keep it for a break-glass group). Verify the policy took effect by checking what methods Alex can register.

**You'll be able to:** Configure the Authentication Methods Policy for a real organization. Know which methods to enable and which to restrict.

---

### 3.3 — Setting Up MFA
**You will learn:** The difference between per-user MFA (old, deprecated) and CA-based MFA (correct, current). How to register authentication methods as a user. How to check MFA registration status across your tenant.

**Lab:** Register Microsoft Authenticator for Alex Lee — go through the complete registration flow as a user. Check MFA registration status for all Contoso users using PowerShell. Find the users with no MFA registered.

**You'll be able to:** Check MFA registration coverage across a tenant. Guide a user through registering MFA.

---

### 3.4 — SSPR — Self-Service Password Reset
**You will learn:** What SSPR is, how to configure it, what registration methods are available, how SSPR writeback works for hybrid users, and what happens when SSPR isn't configured (helpdesk ticket flood).

**Lab:** Enable SSPR for all users. Configure required number of methods to 2. As Alex Lee, register SSPR methods at `aka.ms/ssprsetup`. Test SSPR at `aka.ms/sspr` — actually reset your own test password. Find the SSPR event in the audit log.

**You'll be able to:** Configure SSPR for an organization. Verify it's working. Explain to a manager why enabling SSPR reduces helpdesk tickets.

---

### 3.5 — Passwordless Authentication
**You will learn:** What passwordless means and what it doesn't mean, the three Microsoft passwordless options (FIDO2 keys, Windows Hello for Business, Authenticator app), which ones are phishing-resistant and which ones aren't, and how to deploy them.

**Lab:** Configure the Authenticator app for passwordless sign-in for Alex Lee. Go through the complete passwordless sign-in experience. Compare the sign-in log entry for passwordless vs password sign-in. Understand what makes FIDO2 different from the Authenticator app.

**You'll be able to:** Configure passwordless authentication for users. Explain the phishing-resistance difference between methods.

---

### 3.6 — Token Lifetime, Session Management, and Continuous Access Evaluation
**You will learn:** How long access tokens, refresh tokens, and session cookies last by default and why this matters when you make policy changes. How Sign-In Frequency in Conditional Access forces re-authentication after a set period. What Continuous Access Evaluation (CAE) is — how Entra ID can revoke a valid token in near-real-time when conditions change (account disabled, password changed, session revoked). How persistent browser sessions work and how to control them.

**Lab:** Sign in as Alex Lee. Revoke Alex's sessions in the Entra admin center (Identity → Users → Alex Lee → Revoke sessions). Open a new tab — attempt to access a resource. Watch the forced re-authentication. Configure a Sign-In Frequency Conditional Access policy for SharePoint requiring re-authentication every 8 hours. Sign in as Alex and verify the session frequency behavior in the sign-in log. Check the CAE-capable indicator on a sign-in log entry.

**You'll be able to:** Revoke active user sessions. Configure sign-in frequency policies. Explain why a user might still be signed in after you revoke their sessions — and why CAE closes that gap.

---

### 3.7 — Lab: Secure Your Tenant's Authentication — End to End
**This is a full scenario lab combining everything from Unit 3.**

**Scenario:** *Contoso's security team has flagged that users are signing in with passwords only, SMS is enabled (phishing risk), and SSPR is not configured. Fix all of it.*

**What you'll do:**
- Audit current authentication method registrations across all users
- Disable SMS in the Authentication Methods Policy
- Enable Authenticator with number matching required
- Enable SSPR with two required methods
- Verify every Contoso user has MFA registered — remediate those who don't
- Produce a report showing MFA coverage before and after

**You'll be able to:** Assess authentication security posture, find gaps, and fix them.

---

---

## UNIT 4 — Conditional Access
*The policy engine that makes access context-aware. The most important skill in this series.*

> **SC-300 mapping:** Domain 2 — largest single skill area in the exam
> **Estimated time:** 9–11 hours

---

### 4.1 — Security Defaults: Understanding What You're Replacing
**You will learn:** What Security Defaults are — the baseline set of identity security controls Microsoft enables automatically on new tenants. What they enforce (require MFA for all users, require MFA for administrators, block legacy authentication protocols). Why Security Defaults and Conditional Access policies cannot coexist — you must disable Security Defaults before building your own CA policies. How to check whether they are currently enabled, and how to disable them safely.

**Lab:** Navigate to Identity → Overview → Properties → Manage Security Defaults. Check whether Security Defaults are enabled in your dev tenant. Before disabling: create your first emergency access account (a cloud-only account with a FIDO2 key or long random password, excluded from all policies). Create a break-glass exclusion group and add the emergency account to it. Now disable Security Defaults. Verify they are off. Review exactly what is no longer protecting your tenant — document the three things Security Defaults were enforcing that you will rebuild in the next seven modules.

**You'll be able to:** Check Security Defaults status, disable them safely with emergency access in place, and explain exactly what protection you removed and must replace with CA policies.

---

### 4.2 — What Conditional Access Is — The IF-THEN Engine
**You will learn:** The core IF-THEN model: if these conditions are met, enforce these controls. How multiple policies interact on the same sign-in. Why the result is the most restrictive combination of all applicable policies.

**Lab:** Explore the Conditional Access blade. Find the What-If tool. Run a simulation: "If Alex Lee signs in to SharePoint from an unmanaged device outside the UK — what happens?" (Answer with no policies: nothing. That's the problem you'll fix.)

**You'll be able to:** Explain how Conditional Access works to someone who has never used it. Use the What-If tool to predict policy outcomes.

---

### 4.3 — Your First Policy — Require MFA for Administrators
**You will learn:** How to build a Conditional Access policy from scratch — assignments, conditions, grant controls. How Report-Only mode works and why it's mandatory before enforcement. How to read the CA result in the sign-in log.

**Lab:** Create a policy: all users in the Global Administrator role must satisfy MFA for all apps. Deploy in Report-Only first. Sign in as your admin account. Find the sign-in log and read what the report-only policy would have done. Switch to Enabled. Sign in again — experience the MFA prompt.

**You'll be able to:** Build, test, and enable your first Conditional Access policy. Read CA results in sign-in logs.

---

### 4.4 — Named Locations — Trusted and Untrusted Networks
**You will learn:** What Named Locations are, the difference between IP-range locations and country locations, how to mark a location as trusted, and how to use locations as a CA condition.

**Lab:** Create a Named Location using your home IP address and mark it as trusted. Create a country-based Named Location blocking sign-ins from specific high-risk countries. Build a CA policy that enforces stricter controls outside trusted locations. Test from inside and outside the trusted location.

**You'll be able to:** Configure Named Locations and use them in CA policies. Explain the difference between trusted and non-trusted locations.

---

### 4.5 — Device Conditions — Managed vs Unmanaged
**You will learn:** How CA evaluates device state, the difference between requiring a compliant device vs requiring a domain-joined device, why device compliance is stronger than MFA alone against token theft attacks.

**Lab:** Build a CA policy in Report-Only mode requiring a compliant device for SharePoint access. Sign in as Alex from a managed device — check what the log shows. Sign in from an unmanaged device (incognito, different browser profile) — see the difference in the log.

**You'll be able to:** Configure device-based CA conditions. Explain to a security team why device compliance stops attacks that MFA doesn't.

---

### 4.6 — App Conditions — Not All Apps Are Equal
**You will learn:** How to scope CA policies to specific apps vs all apps, why some apps need stricter policies than others, how to use client app conditions to block legacy authentication protocols.

**Lab:** Create a policy scoped to only the Azure portal and Azure management apps — require phishing-resistant MFA (authentication strength). Create a separate policy scoped to all other apps requiring standard MFA. Verify the different experiences when signing in to the Azure portal vs Teams.

**You'll be able to:** Apply different CA policies to different applications based on their sensitivity.

---

### 4.7 — Report-Only Mode and Testing Safely
**You will learn:** Why Report-Only mode is not optional — it's how you test without consequences. How to read report-only results in sign-in logs. How to use the What-If tool to simulate policy impact before enabling.

**Lab:** Take an existing Report-Only policy. Use the What-If tool to simulate 5 different user/app/device/location combinations and predict the outcome. Check the actual sign-in logs against your predictions. Find at least one case where the policy would do something unexpected.

**You'll be able to:** Use Report-Only mode and the What-If tool to test CA policies safely before enforcement.

---

### 4.8 — Lab: Build a Production-Ready CA Policy Stack
**This is the most important lab in the series. Build a complete, real-world CA policy stack for Contoso.**

**The policy stack you will build:**

| # | Policy | Scope | Control |
|---|---|---|---|
| 1 | Block legacy authentication | All users, All apps | Block |
| 2 | Require MFA for administrators | Admin roles | Phishing-resistant MFA |
| 3 | Require MFA for all users | All users, All apps | MFA |
| 4 | Require compliant device for sensitive apps | All users, SharePoint + Exchange | Compliant device |
| 5 | Block risky sign-ins | All users, All apps | Block (high sign-in risk) |
| 6 | Require password change for risky users | All users, All apps | Require password change (high user risk) |
| 7 | Block unknown countries | All users, All apps | Block (outside named locations) |

**Rules you must follow:**
- Every policy starts in Report-Only
- Test each with What-If before enabling
- Emergency access accounts must be excluded from all block policies
- Document each policy with what it does and why

> **Note:** Policies 5 and 6 require Identity Protection (covered in Unit 6). Configure them in Report-Only now and enable them after completing Unit 6.

**You'll be able to:** Design and deploy a complete CA policy stack. Know why each policy exists and what it stops.

---

---

## UNIT 5 — Applications and SSO
*Connect every app your organization uses to Entra ID for single sign-on.*

> **SC-300 mapping:** Domain 2 — Implement access management for applications
> **Estimated time:** 8–9 hours

---

### 5.1 — The App Integration Big Picture
**You will learn:** What it means to "connect an app to Entra ID," the difference between gallery apps and custom apps, the two objects that every app integration creates (App Registration + Service Principal/Enterprise App), why these are two separate objects.

**Lab:** Browse the Microsoft Entra application gallery. Find three gallery apps relevant to a typical organization. Look at an existing Microsoft service (Teams) in Enterprise Applications — find its service principal and explore its properties.

**You'll be able to:** Explain what app integration means. Find and explore existing app integrations in Enterprise Applications.

---

### 5.2 — Enterprise Applications vs App Registrations
**You will learn:** The App Registration is the global definition (owned by the developer). The Enterprise Application (Service Principal) is the local instance in your tenant (where YOUR governance happens). Admin consent, user assignments, and SSO are all configured on the Enterprise Application.

**Lab:** Register a new app in App Registrations. Find the Enterprise Application that was automatically created in your tenant. Compare what lives on each — see why the permission granted in your tenant is on the Enterprise App, not the App Registration.

**You'll be able to:** Navigate between App Registration and Enterprise Application for the same app. Know which object to change when configuring SSO vs permissions.

---

### 5.3 — Adding a Gallery App and First SSO
**You will learn:** How to add a pre-integrated gallery app, what "SSO" means in the context of a SaaS app, how Entra ID acts as the Identity Provider, how the app is configured to trust Entra ID.

**Lab:** Add a gallery app that supports SSO testing (use the "Contoso" demo app or any available gallery app with SSO). Configure basic SSO. Assign Alex Lee access. Test sign-in as Alex — experience the SSO flow from the My Apps portal (`myapps.microsoft.com`).

**You'll be able to:** Add any gallery app and configure basic SSO. Assign users and test the sign-in experience.

---

### 5.4 — SAML SSO — Step by Step
**You will learn:** How SAML SSO works — Entra ID as the Identity Provider, the app as the Service Provider, what a SAML assertion contains, the four things you must configure (Entity ID, ACS URL, signing certificate, attribute claims).

**Lab:** Configure SAML SSO for a gallery app that supports SAML. Download the federation metadata. Configure the four required fields. Map attribute claims (email, name, groups). Test the SSO flow. Inspect the SAML assertion in browser developer tools to see what Entra ID sent.

**You'll be able to:** Configure SAML SSO for any enterprise application. Diagnose common SAML configuration errors.

---

### 5.5 — OIDC/OAuth SSO — The Modern Approach
**You will learn:** How OIDC SSO works, why it's simpler than SAML for modern apps, the redirect URI requirement, what the authorization code flow looks like in practice, when to use OIDC vs SAML.

**Lab:** Register a custom app for OIDC SSO. Configure redirect URIs. Use Graph Explorer to get an ID token for a user. Decode the token at jwt.ms and identify the user claims. Compare the claims in an OIDC token vs a SAML assertion.

**You'll be able to:** Configure OIDC SSO for modern applications. Choose between OIDC and SAML for a new app integration.

---

### 5.6 — User Assignment — Control Who Can Access an App
**You will learn:** How user assignment controls who can sign in to an app, the difference between open access (anyone in the tenant can use the app) and assignment required (only explicitly assigned users), how to assign users and groups to an app.

**Lab:** Enable "Assignment required" on an Enterprise Application. Try to sign in as a user who isn't assigned — experience the access denied error. Assign Alex to the app — now Alex can sign in. Assign a group instead of individual users.

**You'll be able to:** Control app access at the user/group level. Explain why assignment required is important for sensitive apps.

---

### 5.7 — App Permissions and Admin Consent
**You will learn:** The difference between delegated permissions (app acts on behalf of user) and application permissions (app acts as itself), what admin consent means and why it's tenant-wide, how to review what permissions apps have been granted, why over-permissioned apps are a security risk.

**Lab:** Open Enterprise Applications → Permissions. Find an app with broad permissions. Review what it has been granted and when. Use PowerShell to list all apps with tenant-wide admin consent. Identify any apps with `User.ReadWrite.All` or similarly broad permissions.

**You'll be able to:** Audit app permissions across a tenant. Explain the difference between delegated and application permissions.

---

### 5.8 — Automated Provisioning with SCIM
**You will learn:** What SCIM provisioning is, how it automatically creates user accounts in downstream apps, how attribute mappings work, how to monitor provisioning and find errors.

**Lab:** Enable provisioning on a gallery app that supports SCIM. Configure the provisioning scope (which users). Map attributes from Entra ID to the app's user schema. Run an initial cycle. Check the provisioning logs — find the users that were created.

**You'll be able to:** Configure SCIM provisioning for SaaS apps. Read provisioning logs and diagnose sync errors.

---

### 5.9 — Consent Settings and the Admin Consent Workflow
**You will learn:** The difference between user consent (each user grants an app permissions for themselves) and admin consent (an admin grants permissions tenant-wide on behalf of all users). The consent settings that control what users are allowed to consent to — and the risks of allowing too much. How to configure the admin consent request workflow so users can request app access without needing admin rights themselves. How to review, approve, and revoke consents.

**Lab:** Navigate to Identity → Applications → Enterprise applications → Consent and permissions. Configure user consent settings: set to "Allow user consent for apps from verified publishers, for selected permissions only." Enable the admin consent request workflow with Jordan Kim as an approver and notification email configured. As Alex Lee, attempt to access an app that requires permissions beyond what the user consent policy allows — trigger the admin consent request. As Jordan Kim, open the pending request notification, review what the app is requesting, and approve it. Verify that the app now has tenant-wide admin consent in the Enterprise Applications → Permissions blade.

**You'll be able to:** Configure user consent settings to limit what users can approve themselves. Enable and manage the admin consent request workflow. Review and revoke existing app consents across the tenant.

---

---

## UNIT 6 — Identity Security
*Detect attacks. Investigate compromised accounts. Respond before damage is done.*

> **SC-300 mapping:** Domain 2 — risk-based access management
> **Estimated time:** 5–6 hours

---

### 6.1 — How Identity Attacks Work
**You will learn:** The most common attacks against Entra ID accounts — password spray, phishing, AiTM (adversary-in-the-middle), MFA fatigue, leaked credentials, token theft. How each attack works at the technical level and what stops each one.

**Lab:** Work through each attack scenario using sign-in logs and Graph Explorer. For each attack type: query the sign-in logs for the signals that would indicate that attack (for example, multiple failed sign-ins with different passwords = password spray pattern; anonymous IP + unfamiliar location = potential token theft). Build your own attack-to-defense reference table: attack method, what it looks like in the logs, which Entra ID control stops it, and why.

**You'll be able to:** Identify the log signals for each major identity attack type. Map each attack to the specific Entra ID control that mitigates it.

---

### 6.2 — Identity Protection — Detecting Risky Sign-Ins
**You will learn:** What Identity Protection is, how risk detections work (machine learning on sign-in signals), the difference between sign-in risk (this specific sign-in looks suspicious) and user risk (this account is probably compromised), how to view risk reports.

**Lab:** Navigate to Identity Protection → Risk detections, Risky sign-ins, Risky users. Trigger a risk detection: sign in using Tor Browser or a VPN exit node — Entra ID will detect the anonymous IP. Find the detection in the risk reports.

**You'll be able to:** Navigate Identity Protection reports. Understand the difference between sign-in risk and user risk.

---

### 6.3 — Investigating and Remediating Risky Users
**You will learn:** How to investigate a risky user — what to look at, what questions to ask, how to determine if the account is actually compromised. The two remediation paths: confirm compromised vs dismiss risk as safe.

**Lab:** Find a risky user in Identity Protection. Click through the full investigation workflow: view their recent sign-ins, look for anomalies, check what risk detections fired. Make a remediation decision. Confirm as compromised (triggers forced password reset). Then dismiss as safe and see what that does.

**You'll be able to:** Investigate a risky user report and make a correct remediation decision.

---

### 6.4 — Risk-Based Conditional Access
**You will learn:** How to build CA policies that respond to risk in real time — block high-risk sign-ins, require password reset for high-risk users, step up to MFA for medium-risk sign-ins.

**Lab:** Create two risk-based CA policies:
- IF sign-in risk = High → Block access
- IF user risk = High → Require password change

Test both policies. Trigger a high sign-in risk event and watch the policy block it. Trigger a high user risk event and watch the forced password reset.

**You'll be able to:** Build and test risk-based Conditional Access policies.

---

### 6.5 — Blocking Legacy Authentication
**You will learn:** What legacy authentication protocols are (SMTP AUTH, POP3, IMAP, older Office clients), why they're dangerous (they can't do MFA), how to identify if legacy auth is being used in your tenant, how to block it.

**Lab:** Run the sign-in log query to find any legacy authentication sign-ins in your tenant. Build a CA policy to block all legacy authentication client types. Put in Report-Only, check the impact. Enable enforcement. Verify legacy auth is now blocked.

**You'll be able to:** Identify legacy auth usage and block it. Explain why blocking legacy auth is one of the highest-impact security improvements.

---

### 6.6 — Lab: Account Compromise Response
**This is a full scenario lab.**

**Scenario:** *At 11pm, an alert fires: Jordan Kim (IT Administrator) has been flagged as a High risk user. Their account shows sign-ins from an anonymous IP in a country they've never accessed from. You are the on-call admin.*

**What you'll do:**
- Immediately revoke all active sessions for Jordan's account
- Review Jordan's complete sign-in history
- Check what apps Jordan accessed and from where
- Review audit logs for admin actions taken by Jordan in the last 24 hours
- Confirm the account as compromised
- Reset Jordan's password and force re-registration of MFA
- Document your investigation and response timeline

**You'll be able to:** Respond to an account compromise incident step by step.

---

---

## UNIT 7 — Privileged Access
*Protect your most powerful accounts. Eliminate standing admin access.*

> **SC-300 mapping:** Domain 4 — Plan and automate identity governance
> **Estimated time:** 6–7 hours

---

### 7.1 — The Danger of Standing Privileged Access
**You will learn:** What standing access means and why it's dangerous, the blast radius of a compromised Global Administrator account, what "least privilege" means in practice, and why time-limited access is better than permanent access.

**Lab:** Audit your current privileged role assignments. Find which accounts have permanent Global Administrator access. Calculate the blast radius: what can a compromised Global Admin do in your tenant? This audit will motivate why PIM matters.

**You'll be able to:** Audit privileged role assignments and explain the security risk of standing access.

---

### 7.2 — Understanding Entra ID Roles
**You will learn:** How built-in directory roles work, the most important roles and what they can do, the principle of least-privilege role assignment, why Global Administrator should almost never be permanently assigned.

**Lab:** Review the built-in roles list. For each Contoso character, identify the minimum role they need for their job. Compare Global Administrator vs User Administrator vs Helpdesk Administrator — what can each do and what can't they do?

**You'll be able to:** Choose the right Entra ID role for any admin task. Explain why least-privilege role assignment matters.

---

### 7.3 — Setting Up PIM — Privileged Identity Management
**You will learn:** What PIM is, how it converts permanent admin assignments to eligible assignments, what the activation workflow looks like, how to configure PIM for Entra ID roles.

**Lab:** Enable PIM in your tenant. Convert your own Global Administrator assignment from permanent to eligible. Configure the role settings: require MFA on activation, require justification, set a 4-hour maximum activation duration.

**You'll be able to:** Enable PIM and convert permanent role assignments to eligible.

---

### 7.4 — Just-in-Time Activation — The Workflow
**You will learn:** The complete JIT activation flow from a user's perspective — how to request activation, what happens during approval, how the time-limited access works, what happens when it expires.

**Lab:** As Jordan Kim (your IT admin character): activate the Global Administrator role via PIM. Experience the full activation flow — MFA prompt, justification entry, activation confirmation. Use the admin access for a task. Watch it expire. Try to use admin access after expiry — see the error.

**You'll be able to:** Activate a PIM role and explain the complete JIT workflow.

---

### 7.5 — PIM Approval Workflows
**You will learn:** How to configure PIM roles to require approval before activation, who can approve activation requests, how approvers receive and act on requests, how to audit PIM activations.

**Lab:** Configure the User Administrator role to require approval by Jordan Kim. As Alex Lee, request User Administrator activation. As Jordan Kim, receive the approval notification and approve it. Check the PIM audit log — find every activation event.

**You'll be able to:** Configure PIM approval workflows and process approval requests.

---

### 7.6 — Lab: Emergency Access — When Normal Access Fails
**This is a full scenario lab.**

**Scenario:** *Your MFA service has an outage. All your Global Admin accounts require MFA to activate in PIM. You're locked out of your own tenant. This is one of the most catastrophic scenarios in identity management.*

**What you'll do:**
- Create two emergency access accounts following Microsoft's official guidance
- Configure them correctly: cloud-only, FIDO2 key, excluded from all CA policies
- Verify they work: sign in as the emergency account, confirm access
- Document the emergency access procedure (where are credentials stored? who knows?)
- Understand why these accounts exist and why they must never be used for normal admin tasks

**You'll be able to:** Set up emergency access accounts correctly and explain their purpose.

---

---

## UNIT 8 — Identity Governance
*Ensure people have the right access — and that it doesn't accumulate forever.*

> **SC-300 mapping:** Domain 4 — Plan and automate identity governance
> **Estimated time:** 7–8 hours

---

### 8.1 — The Joiner-Mover-Leaver Model
**You will learn:** The three lifecycle events that define identity governance: someone joining the organization, someone changing roles, someone leaving. Why most organizations handle joining reasonably well but fail at movers and leavers. The access accumulation problem.

**Lab:** Audit the Contoso identities you've built so far. Which ones have more access than their role requires? This audit reveals the access accumulation problem that governance solves.

**You'll be able to:** Explain the joiner-mover-leaver model and identify access accumulation in a tenant.

---

### 8.2 — Entitlement Management — Access Packages
**You will learn:** What access packages are, how they bundle resources (groups, apps, SharePoint sites) into a requestable unit, how the approval workflow works, how automatic expiry keeps access from persisting forever.

**Lab:** Create an Access Package called "Finance Team Access" that includes: the Finance security group, a SharePoint site, and an app. Configure an approval workflow requiring River Patel (HR) to approve. Set access to expire after 90 days. As Alex Lee, request the package via myaccess.microsoft.com.

**You'll be able to:** Create and configure an Access Package end to end.

---

### 8.3 — Access Reviews — Certifying Access Over Time
**You will learn:** What access reviews are and why they matter for compliance, the types of reviews (group membership, app assignment, role assignment), how to create a review, how the reviewer experience works, what happens when reviewers don't respond.

**Lab:** Create an access review for the Finance security group. Make River Patel (manager) the reviewer. Complete the review process: receive the email notification, review each member, approve two and deny one. See the denied user automatically removed.

**You'll be able to:** Create and manage access reviews. Configure auto-remediation for non-responses.

---

### 8.4 — Lifecycle Workflows — Automating the Lifecycle
**You will learn:** What Lifecycle Workflows are, the built-in workflow templates (Joiner, Leaver), how to trigger workflows from HR attributes (start date, last day), what tasks can be automated (send welcome email, add to groups, remove access, disable account).

**Lab:** Create a Joiner workflow triggered by a future start date attribute. Set tasks: add user to onboarding group, send a welcome email, enable account on start date. Create a Leaver workflow triggered by last day attribute. Set tasks: remove all group memberships, disable account, revoke sessions. Test both workflows with the Contoso characters.

**You'll be able to:** Build automated joiner and leaver workflows using Lifecycle Workflows.

---

### 8.5 — Lab: The Contractor Whose Access Never Expired
**This is a full scenario lab.**

**Scenario:** *Sam Taylor was a contractor who joined 18 months ago for a 6-month project. The project ended, Sam left — but no one offboarded them. Sam still has active access to SharePoint, two applications, and is a member of the Engineering group. Your job: find the problem, understand how it happened, fix it, and put controls in place so it can't happen again.*

**What you'll do:**
- Audit Sam's current access and find everything they have
- Immediately revoke Sam's access (disable, revoke sessions, remove memberships)
- Identify the process gap that allowed this to happen
- Configure an Access Package with 90-day expiry and access review so it can't happen again
- Configure a Lifecycle Workflow that would have automated the offboarding

**You'll be able to:** Identify over-permissioned users, remediate stale access, and build controls to prevent recurrence.

---

---

## UNIT 9 — Devices and Compliance
*Make "what device you're on" part of every access decision.*

> **SC-300 mapping:** Domain 2 — device-based access controls
> **Estimated time:** 6–7 hours

---

### 9.1 — Device Identity in Entra ID
**You will learn:** What a device identity is, the three device join states (Entra Registered, Entra Joined, Hybrid Entra Joined), what each state means in terms of management and trust, how devices appear in Entra ID.

**Lab:** Find the Devices section in Entra ID. Look at what devices currently exist. Find your own device. Identify its join state, compliance status, and management state. Understand what information Entra ID has about each device.

**You'll be able to:** Read a device object in Entra ID and identify its state, trust level, and management status.

---

### 9.2 — Device Registration, Join, and Hybrid Join — What's the Difference
**You will learn:** Entra Registered (personal device, BYOD — light management), Entra Joined (corporate cloud device — full management), Hybrid Entra Joined (corporate on-prem domain joined AND Entra ID — for existing AD environments). When to use each.

**Lab:** Entra-register your personal device to your dev tenant (Settings → Accounts → Add work or school account). Find the device in Entra ID. Explore what Entra can and cannot control about a registered (not joined) device.

**You'll be able to:** Register a device and explain the difference between registration, join, and hybrid join.

---

### 9.3 — Intune Enrollment — Adding Management
**You will learn:** What Intune enrollment adds on top of device registration, how enrollment allows pushing policies, compliance checks, and app deployment, how to enroll a device in Intune.

**Lab:** Enroll your device in Intune (via the Company Portal app or MDM enrollment settings). Verify the device appears in the Intune portal. See what information Intune collects about the device. Compare: the same device before and after enrollment — what changed in Entra ID?

**You'll be able to:** Enroll a device in Intune. Explain what management capabilities enrollment enables.

---

### 9.4 — Compliance Policies — Defining What "Secure" Means
**You will learn:** How to create a compliance policy in Intune, what requirements you can set (OS version, BitLocker, screen lock, firewall, jailbreak detection), how compliance status flows from Intune to Entra ID, and how long compliance evaluation takes.

**Lab:** Create a compliance policy for Windows: require BitLocker, require minimum OS version, require screen lock. Assign it to your enrolled device. Check the compliance status in Intune and in Entra ID. Deliberately fail a compliance requirement — watch the device become non-compliant. Remediate and watch it become compliant again.

**You'll be able to:** Create compliance policies and trace compliance status through Intune to Entra ID.

---

### 9.5 — Device-Based Conditional Access
**You will learn:** How to require a compliant device in a CA policy, what happens to users whose devices aren't compliant, how to combine device compliance with other CA controls, and why compliant device requirements stop attacks that MFA doesn't.

**Lab:** Enable the CA policy requiring compliant device for SharePoint (built in Unit 4, Module 4.5 — was in Report-Only). Now that you have a compliant device, test it: sign in from your compliant device (allowed), sign in from an incognito browser simulating an unmanaged device (blocked). Read the CA result in the sign-in log for each.

**You'll be able to:** Enable and test device compliance CA policies. Read the device compliance result in sign-in logs.

---

### 9.6 — App Protection Policies — BYOD Without Full Management
**You will learn:** What App Protection Policies (MAM) do — protect corporate data inside specific apps without enrolling the device, how to configure them for Outlook and Teams on mobile, when MAM is the right choice vs full MDM.

**Lab:** Create an App Protection Policy for iOS/Android Outlook: require PIN, block copy-paste to unmanaged apps, block saving attachments to personal storage. Assign it to the Finance group. Test the policy behavior in Outlook on a personal mobile device (your phone).

**You'll be able to:** Configure App Protection Policies for mobile apps. Explain the BYOD scenario where MAM is better than MDM.

---

### 9.7 — Lab: BYOD — Personal Devices, Company Data
**This is a full scenario lab.**

**Scenario:** *Contoso has a new BYOD policy. Employees can use personal devices for work, but company data must be protected. You must implement a solution that: protects data without requiring full device management, lets employees use Outlook and Teams on their phones, blocks access to sensitive apps from completely unmanaged devices.*

**What you'll build:**
- App Protection Policies for Outlook and Teams on mobile
- CA policy: sensitive apps require either compliant device OR app protection policy
- User communication: what employees need to do to access work apps on their phone
- Test the complete flow as a user with a personal device

**You'll be able to:** Design and implement a BYOD strategy using App Protection Policies and CA.

---

---

## UNIT 10 — Hybrid Identity
*Bring your on-premises Active Directory into the cloud.*

> **SC-300 mapping:** Domain 1 — implement and manage hybrid identity
> **Estimated time:** 8–9 hours (includes Azure VM lab setup)

---

### 10.1 — Entra Connect Sync vs Cloud Sync: Choosing Your Sync Engine
**You will learn:** Microsoft provides two sync engines for connecting on-premises AD to Entra ID. Entra Connect Sync (formerly Azure AD Connect) is the older, full-featured tool — installed on a dedicated server, supports complex topologies, writeback, and legacy scenarios. Entra Cloud Sync is the newer, lightweight agent-based approach — no dedicated server needed, simpler to deploy, but with fewer advanced capabilities. Most enterprises you work in will have Entra Connect Sync already running. New greenfield deployments and simpler environments are moving to Cloud Sync. You need to recognize both, know their differences, and recommend the right one.

**Lab:** Planning exercise — you are given three organization scenarios. For each one, choose the sync engine and justify the decision in writing:
1. A 40-person company with no existing on-prem AD infrastructure, moving fully to Microsoft 365.
2. A 3,000-person enterprise running Entra Connect Sync with password writeback, device writeback, and Exchange hybrid configured.
3. A 200-person company currently using ADFS for federation, planning to migrate to Entra ID.
Build a one-page comparison reference table: Cloud Sync vs Entra Connect Sync — supported features, deployment model, management complexity, and when to use each.

**You'll be able to:** Recommend the right sync engine for any organization scenario. Explain the key differences between Cloud Sync and Entra Connect Sync to a manager or architect.

---

### 10.2 — Why Hybrid Identity Exists
**You will learn:** The business reality — most enterprises have years of on-premises AD. The problem hybrid identity solves. The architectural decision: cloud-only vs hybrid. What you gain and lose with each approach.

**Lab:** No configuration lab. Audit your dev tenant: is it cloud-only? Now map out a fictional organization that has 500 users in on-prem AD and is moving to Microsoft 365. What decisions do they need to make? This planning exercise sets up the rest of Unit 10.

**You'll be able to:** Explain what hybrid identity is and when an organization needs it.

---

### 10.3 — How Directory Sync Works
**You will learn:** The sync engine, what gets synced (users, groups, contacts, devices), what doesn't sync by default, what writeback means, the direction of sync (mostly one-way), the sync cycle and frequency.

**Lab:** Set up a Windows Server VM in Azure with AD DS (step-by-step guided). Install Entra Cloud Sync agent. Create users in on-prem AD — watch them appear in Entra ID. Change an attribute in Entra ID — watch it get overwritten by the next sync. Change it in AD — watch it propagate correctly.

**You'll be able to:** Install Entra Cloud Sync and understand which direction changes flow.

---

### 10.4 — Authentication Methods — PHS, PTA, and Federation
**You will learn:** Password Hash Sync (hashed passwords sync to cloud — most resilient), Pass-Through Authentication (auth forwarded to on-prem AD in real time — most dependencies), Federation (all auth delegated to on-prem IdP — most complex). When to choose each.

**Lab:** Configure Password Hash Sync in your Cloud Sync setup. Create a user in AD, sync them to Entra ID, and sign in as that user using their AD password in the cloud. Understand what would be different if PTA or Federation were used instead.

**You'll be able to:** Configure PHS and explain the tradeoffs between PHS, PTA, and Federation.

---

### 10.5 — Source of Authority — What to Edit Where
**You will learn:** Source of Authority is the single most important concept for hybrid identity. Synced attributes are owned by on-premises AD — editing them in Entra ID leads to them being overwritten at the next sync cycle. Cloud-only attributes can be managed in Entra ID.

**Lab:** Identify which attributes on a synced user are AD-owned vs cloud-owned. Try to change the `department` attribute in Entra ID — watch the sync overwrite it. Change it in AD — watch it propagate. Find the list of attributes that are safe to manage in Entra ID for synced users.

**You'll be able to:** Identify where to make attribute changes for synced users. Explain why editing synced attributes in the cloud doesn't work.

---

### 10.6 — Troubleshooting Sync Errors
**You will learn:** How to read provisioning logs for sync errors, the most common sync errors and what causes them (duplicate attributes, invalid UPN format, missing required attributes), how to fix each one.

**Lab:** Deliberately cause three common sync errors: create a user in AD with a duplicate UPN, create a user with an invalid email format, create a user without a required attribute. Find each error in the provisioning logs. Fix each one in AD and verify the sync succeeds.

**You'll be able to:** Read sync error logs and fix the five most common sync failures.

---

### 10.7 — Lab: The Sync Scenario
**This is a full scenario lab.**

**Scenario:** *A company has 200 users in on-prem Active Directory. They are migrating to Microsoft 365. You must: set up sync, get all users to Entra ID, configure authentication, ensure attributes are correct, and verify sign-in works for a sample of users.*

**What you'll do:**
- Set up Cloud Sync for your test AD environment
- Sync all test users to Entra ID
- Verify the sync results and fix any errors
- Test sign-in for three synced users
- Configure SSPR writeback so users can reset their on-prem password from the cloud
- Produce a sync health report

**You'll be able to:** Plan and execute a directory sync implementation.

---

---

## UNIT 11 — Workload Identities
*Secure apps and automation — without secrets stored in code or config files.*

> **SC-300 mapping:** Domain 3 — Plan and implement workload identities
> **Estimated time:** 6–7 hours

---

### 11.1 — The Problem With Passwords in Code
**You will learn:** What workload identities are (non-human identities for apps, scripts, pipelines), why storing client secrets in code is dangerous, what happens when a secret leaks, what the alternatives are.

**Lab:** Audit your dev tenant for app registrations with active client secrets. Find their expiry dates. Identify any secrets that are expired or overdue for rotation. This audit reveals the credential management problem that managed identities solve.

**You'll be able to:** Audit workload credentials in a tenant and identify risks.

---

### 11.2 — App Registrations — Registering Your Application
**You will learn:** How to register an application in Entra ID, what an App Registration contains (redirect URIs, API permissions, certificates and secrets), how to use the app to authenticate to Microsoft Graph.

**Lab:** Register a new application called "Contoso Automation." Add Microsoft Graph API permissions (User.Read.All, Group.Read.All). Grant admin consent. Create a client secret. Use the secret to authenticate to Microsoft Graph and call the users API via PowerShell — see Alex Lee's profile returned.

**You'll be able to:** Register an app, configure API permissions, and use it to call Microsoft Graph.

---

### 11.3 — Managing App Credentials — Secrets and Certificates
**You will learn:** The difference between client secrets (simple, less secure) and certificates (more complex, more secure), secret expiry management, why Microsoft recommends certificates over secrets for production apps.

**Lab:** Add a certificate to your app registration (self-signed for the lab). Authenticate using the certificate instead of the secret. Compare: how is certificate authentication different from secret authentication? Configure a secret expiry alert.

**You'll be able to:** Configure both secrets and certificates for app authentication. Explain why certificates are preferred.

---

### 11.4 — Managed Identities — No Credentials at All
**You will learn:** What managed identities are, how Azure assigns and manages the identity so you never handle a credential, the difference between system-assigned and user-assigned managed identity, how the Instance Metadata Service works.

**Lab:** Create an Azure Function App (or App Service). Enable a system-assigned managed identity. Grant the managed identity "Reader" access on a resource group. Write a simple function that calls Azure Management API using the managed identity token — without any credential in the code. Verify it works.

**You'll be able to:** Configure a managed identity and use it to access Azure resources without credentials.

---

### 11.5 — Workload Identity Federation — External Workloads Without Secrets
**You will learn:** What Workload Identity Federation is and the problem it solves — external workloads (CI/CD pipelines, cloud services) can authenticate to Entra ID using their own OIDC tokens instead of storing a client secret. How GitHub Actions, GitLab, Google Cloud, AWS, and Kubernetes can all use this pattern. How federated credentials work at the technical level.

**Lab:** Create an app registration for a GitHub Actions integration. Add a federated credential pointing to your GitHub repository. Configure a GitHub Actions workflow using `azure/login@v2` with OIDC (no secrets stored in GitHub). Run the workflow — watch it authenticate to Entra ID and call Microsoft Graph without any secret in GitHub Secrets or any code. Verify the sign-in in Entra ID logs.

**You'll be able to:** Configure Workload Identity Federation for GitHub Actions. Explain why the same pattern applies to other CI/CD platforms.

---

### 11.6 — Lab: The Expired Secret Incident
**This is a full scenario lab.**

**Scenario:** *Monday morning, a business-critical automation script stops working. The error: `AADSTS7000215: Invalid client secret provided.` The client secret for the Contoso Automation app expired over the weekend. The script runs hourly and pushes data to SharePoint. It's been failing since 2am.*

**What you'll do:**
- Diagnose the failure from the error message and sign-in logs
- Rotate the client secret (create a new one without deleting the old one first — why does order matter?)
- Update the script with the new secret — test it works
- Implement a certificate instead of a secret to avoid this recurring
- Set up a secret expiry monitoring alert so you know 60 days before next expiry

**You'll be able to:** Respond to a secret expiry incident and implement controls to prevent recurrence.

---

---

## UNIT 12 — Monitoring and Operations
*Read logs fluently. Know what's normal so anomalies stand out. Build runbooks.*

> **SC-300 mapping:** All four domains — operational knowledge is tested throughout
> **Estimated time:** 6–7 hours

---

### 12.1 — Sign-In Logs — Your Primary Diagnostic Tool
**You will learn:** Every field in a sign-in log entry and what it means, how to filter and search effectively, the 20 most common error codes and what causes each, how to export logs for analysis.

**Lab:** Work through 10 real sign-in scenarios — each one is a different failure. Read the log entry for each and diagnose: wrong password, MFA not satisfied, CA policy block, non-compliant device, app not assigned, account disabled, legacy auth attempt, sign-in risk blocked, audience mismatch, expired session. State the exact cause for each within 60 seconds.

**You'll be able to:** Diagnose any sign-in failure from the sign-in log within 2 minutes.

---

### 12.2 — Audit Logs — Who Changed What and When
**You will learn:** What the audit log records, how to search by actor, action type, and date range, how to find who made a specific change, how to export audit logs, retention periods.

**Lab:** Make five administrative changes: create a user, assign a role, create a CA policy, grant admin consent, delete a group. Find each change in the audit log within 2 minutes. Run a query: "Who made any change to Conditional Access policies in the last 7 days?"

**You'll be able to:** Find any administrative change in the audit log. Build audit queries for common compliance questions.

---

### 12.3 — Provisioning Logs — App and Lifecycle Sync Events
**You will learn:** What the provisioning log records, how to read SCIM sync events, how to find why a user wasn't provisioned to a downstream app, how to distinguish provisioning errors from sync errors.

**Lab:** Trigger a provisioning event (create a user who should be provisioned to a connected app). Find the provisioning log entry. Deliberately cause a provisioning failure (missing required attribute). Read the error. Fix the attribute. Watch the user provision successfully on the next cycle.

**You'll be able to:** Read provisioning logs and diagnose sync failures to downstream apps.

---

### 12.4 — Setting Up Alerts — Know Before Users Tell You
**You will learn:** How to configure diagnostic settings to send logs to Log Analytics, how to create alert rules for critical events (admin account sign-in failure, bulk user deletion, new Global Admin assigned, legacy auth attempt).

**Lab:** Configure Diagnostic Settings to send sign-in and audit logs to a Log Analytics workspace. Create an alert rule: fire when a Global Administrator role is assigned to anyone. Trigger the alert by assigning the role. Receive the alert notification.

**You'll be able to:** Configure log routing and create alert rules for critical security events.

---

### 12.5 — Lab: Three Failures to Diagnose
**This is a full scenario lab — three independent incidents.**

**Incident 1:** 50 users can't sign in to Salesforce. The failure started at 9:04am. No changes were made to Salesforce. (`Hint: check CA policy changes in audit log at 9:02am`)

**Incident 2:** A single user can't sign in anywhere. They were working fine yesterday. (`Hint: check the account status, then CA policies, then risk level`)

**Incident 3:** An automated provisioning script stopped creating new users in ServiceNow 3 days ago. No error was reported. (`Hint: check provisioning logs — attribute mapping changed`)

**For each incident:** Find the cause, fix it, verify the fix, explain what happened and how to prevent it.

**You'll be able to:** Diagnose and resolve real operational incidents using Entra ID logs.

---

### 12.6 — The Admin's Weekly Operational Checklist
**You will learn:** What a real Entra ID admin reviews weekly, what proactive monitoring prevents reactive incidents, how to build a personal operational runbook.

**Lab:** Build your own Entra ID operational runbook — a documented procedure for each item in the weekly checklist. For each item: what you check, how you check it, what normal looks like, what to do if it's not normal.

**Weekly checklist covers:** Risky users with no remediation, CA policies in Report-Only that should be enforced, app secrets and SAML certificates near expiry, users with no MFA registered, unexpected new admin role assignments, stale guest accounts, provisioning errors unresolved for 48h+.

**You'll be able to:** Run a proactive weekly admin review and catch problems before users report them.

---

---

## UNIT 13 — Architecture and Expert Design
*Design identity systems. Prepare for certification. Become job-ready.*

> **SC-300 mapping:** All four domains — this unit synthesizes everything
> **Estimated time:** 8–10 hours

---

### 13.1 — Designing a CA Policy Stack from Zero
**You will learn:** The architectural principles for CA policy design — start broad, layer specificity, minimize exclusions, document everything. How to design for a real organization rather than a single policy.

**Lab:** Given a fictional 200-person organization with specific requirements (BYOD policy, remote workforce, sensitive finance data, contractor access), design a complete CA policy stack. Document each policy: what it does, why it exists, what attack it mitigates, what the exclusions are and why.

**You'll be able to:** Design a CA policy stack for a real organization from requirements.

---

### 13.2 — Zero Trust Maturity — Where You Are and How to Advance
**You will learn:** The Zero Trust maturity model, what each stage looks like in practice, how to assess an organization's current maturity, and what the next steps are for each stage.

**Lab:** Use the Microsoft Zero Trust Assessment tool to assess your dev tenant. Identify the gaps. Build a prioritized 90-day improvement plan. Justify the priority order: which gaps carry the most risk if left unaddressed?

**You'll be able to:** Assess Zero Trust maturity and produce an improvement roadmap.

---

### 13.3 — Naming Conventions and Documentation
**You will learn:** Why naming conventions matter at scale, how to name CA policies, groups, access packages, and app registrations consistently, what documentation an Entra ID admin must maintain.

**Lab:** Audit everything you've built in this series — rename everything that doesn't follow a consistent naming convention. Build a documentation template for CA policies that captures: name, purpose, scope, grant controls, exclusions, report-only enabled date, enforcement date, last reviewed date.

**You'll be able to:** Apply naming conventions to Entra ID objects and maintain admin documentation.

---

### 13.4 — Putting It All Together — The Full Contoso Build
**This is the capstone lab. You will build a complete, production-ready Entra ID configuration for Contoso Ltd from scratch.**

**Requirements:**
- 5 employees with correct roles and group memberships
- All users licensed
- MFA required for all users, phishing-resistant for admins
- Full CA policy stack (8 policies)
- Two SaaS apps with SSO and provisioning
- PIM configured for all admin roles
- Access reviews scheduled for all privileged access
- Lifecycle workflows for joiners and leavers
- Emergency access accounts configured and tested
- Device compliance enforced for sensitive apps
- Monitoring and alerts configured
- Everything documented with naming conventions

**You'll be able to:** Build a complete Entra ID configuration for a real organization.

---

### 13.5 — SC-300 Exam Preparation
**You will learn:** The four SC-300 exam domains and their weights, what the exam tests vs what this series covered, the question types and how to approach scenario-based questions, the knowledge gaps most candidates have.

**Lab:** Work through 20 practice scenario questions — one at a time, answer before reading the explanation. For each wrong answer: identify which module you need to review, go back and re-do that lab.

**Current SC-300 exam domains (as of November 2025):**
| Domain | Weight | Key modules |
|---|---|---|
| Implement and manage user identities | 20-25% | Units 2, 10 |
| Implement authentication and access management | 25-30% | Units 3, 4, 6, 9 |
| Plan and implement workload identities | 20-25% | Units 5, 11 |
| Plan and automate identity governance | 20-25% | Units 7, 8 |

**You'll be able to:** Identify your exam readiness and the specific areas to review.

---

### 13.6 — The Job Interview — What Employers Actually Test
**You will learn:** The difference between what job postings say and what interviews actually test, the 10 most common Entra ID interview scenarios, how to demonstrate practical experience without years of enterprise experience, how to talk about your lab environment as real experience.

**Lab:** Answer each of the 10 interview scenarios out loud — time yourself, aim for 90 seconds per answer. Record what you said. Review: did you answer the actual question? Did you give a specific example? Did you mention the tools you used?

**The 10 scenarios covered include:** "Walk me through how you'd respond to a compromised account," "How would you design Conditional Access for a hybrid organization," "A user is blocked — how do you diagnose it?", "What's the difference between PHS and PTA and when would you choose each?"

**You'll be able to:** Answer Entra ID interview questions with specific, confident, experience-backed answers.

---

---

## Learning Resources

**Official Microsoft Documentation:**
- Microsoft Entra ID documentation: `learn.microsoft.com/entra/identity`
- SC-300 exam study guide: `learn.microsoft.com/credentials/certifications/resources/study-guides/sc-300`
- SC-300 exam page: `learn.microsoft.com/credentials/certifications/exams/sc-300`
- Microsoft Learn SC-300 learning path: `learn.microsoft.com/training/paths/implement-manage-hybrid-identity`
- Microsoft Entra changelog: `entra.microsoft.com/changelog` — check monthly to stay current

**Practice Tools:**
- Developer tenant: `developer.microsoft.com/microsoft-365/dev-program`
- Graph Explorer: `aka.ms/ge`
- Token decoder: `jwt.ms`
- My Access portal (user perspective): `myaccess.microsoft.com`
- Zero Trust Assessment: `aka.ms/ztassessment`

**Community Resources:**
- Microsoft Tech Community — Identity blog: `techcommunity.microsoft.com/category/microsoft-entra`
- John Savill's SC-300 Study Cram (YouTube): Search "John Savill SC-300" — the standard community exam prep resource
- Microsoft Entra MVP blog posts: Follow the Entra ID tag on Tech Community for real-world deployment patterns

**Glossary Reference:**
This series was built alongside the **Entra ID Glossary** — 189 term definitions covering every concept in this series. When a module introduces a term and you want the deep-dive definition, the glossary is the reference. Links to the relevant glossary entries appear at the end of each module.

---

> **Entra ID — Zero to Admin** · 13 Units · 86 Modules · Concept + Lab + Scenario
>
> Start at Module 1.1. Work in order. Don't skip labs.
