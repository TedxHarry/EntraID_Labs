# Token Lifetime, Session Management, and Continuous Access Evaluation
*Revoking a user's session does not immediately end their access. Here is why — and how to close that gap.*

> 🟡 **Intermediate** · Unit 3, Module 3.6

---

## What You Will Learn

- State the default lifetime of access tokens, refresh tokens, and browser sessions
- Revoke Alex Lee's active sessions in the Entra admin center and observe the re-authentication behavior
- Configure a Sign-In Frequency Conditional Access policy for SharePoint
- Explain what Continuous Access Evaluation (CAE) is and why it matters for near-real-time revocation
- Find the CAE-capable indicator on a sign-in log entry

---

## Why This Matters

*SC-300: Implement authentication and access management — Implement and manage Microsoft Entra ID session controls*

When you disable a user's account or revoke their access, you expect the change to take effect immediately. It does not — not without CAE. A user with an active access token can continue accessing resources for up to the token's remaining lifetime after their account is disabled. In the default configuration, that window is up to one hour. For a terminated employee or a compromised account, one hour of continued access after the disable action is a significant security gap. Understanding token lifetime and CAE closes that understanding gap — and knowing how to verify CAE is enabled closes the actual security gap.

---

## ⏱️ Default Lifetimes — What to Know Without Looking It Up

| Token / Session | Default lifetime | What happens when it expires |
|---|---|---|
| **Access token** | 1 hour | App requests a new one using the refresh token (silent, no user prompt) |
| **Refresh token (non-persistent)** | 24 hours | User must sign in again |
| **Refresh token (persistent)** | 90 days (slides with use) | User must sign in again after 90 days of inactivity |
| **Browser session cookie** | Until browser closed (non-persistent) or 90 days (persistent) | User must sign in again |
| **Primary Refresh Token (PRT)** | 14 days | Device must re-authenticate |

The **Primary Refresh Token (PRT)** is specific to managed Windows devices. It is a long-lived token that allows seamless SSO across all apps on the device without repeated sign-in prompts. A device with a valid PRT provides a smooth experience — one sign-in to Windows, everything else works.

The critical number: **1 hour** for access tokens. This is the window during which a disabled account or revoked access can still function. Apps that receive an access token do not call Entra ID on every request — they trust the token until it expires.

---

## 🔄 Session Revocation — What It Does and Doesn't Do

When you revoke a user's sessions in the Entra admin center, Entra ID invalidates the refresh tokens. The next time the user's app tries to silently refresh an expired access token using the refresh token, the refresh request fails. The user is then prompted to sign in again.

What revocation does NOT immediately do: it does not terminate active access tokens already in memory. If the user's browser has a valid 1-hour access token to SharePoint that was issued 10 minutes ago, they can continue using SharePoint for another 50 minutes after you revoke their sessions.

This is the gap. Without CAE, session revocation is eventually consistent — not immediate.

---

## ⚡ Continuous Access Evaluation — Near-Real-Time Revocation

Continuous Access Evaluation (CAE) is a protocol where supported resource services (Microsoft Graph, Exchange Online, SharePoint Online, Teams) maintain a persistent connection with Entra ID and receive events that should terminate existing sessions.

When a user's account is disabled, their password is changed, or their sessions are revoked, Entra ID sends a signal to CAE-capable resource services. Those services then require re-authentication the next time the user makes a request — even if their access token has not expired yet.

The re-authentication happens within seconds to minutes, not at the next token expiry.

**Two conditions required for CAE to work:**
1. The resource service must support CAE (Exchange, SharePoint, Teams, Graph do as of 2024)
2. The client application must be CAE-capable (modern Office apps, Microsoft 365 clients, Outlook for Windows/Mac are CAE-capable)

You can see whether a sign-in was CAE-enabled in the sign-in log — the sign-in detail pane shows a **Continuous access evaluation** field.

> 🔑 **Key concept:** CAE does not apply to all apps or all scenarios. Third-party apps that use Graph API indirectly may not receive CAE signals. For a compromised account, disabling the account AND revoking sessions AND confirming CAE coverage on the resources being used is the complete response — not just checking one box.

---

## 🔁 Sign-In Frequency — Forcing Re-Authentication on a Schedule

Sign-In Frequency is a Conditional Access session control that forces users to re-authenticate after a specified period — regardless of whether their tokens have expired.

Use cases:
- SharePoint: require re-authentication every 8 hours for sensitive document repositories
- PIM activation: require re-authentication before activating a privileged role
- Privileged admin portals: require daily re-authentication for admin center access

Sign-In Frequency is not a replacement for CAE — it is a scheduled control. CAE is an event-driven control. Both have a place in a mature security posture.

---

## Lab 3.6.1 — Revoke Alex Lee's Sessions and Observe Re-Authentication

**Tools:** Entra admin center, InPrivate browser
**Prerequisite:** Alex Lee is signed in to a resource (from Module 3.5 lab or a fresh sign-in)

**Step 1: Sign Alex in to Office 365**

1. Open an InPrivate browser. Navigate to `portal.office.com`. Sign in as Alex Lee.

2. Once signed in, leave the portal open — Alex's session is active.

**Step 2: Revoke Alex's sessions from the admin center**

3. In your admin browser, navigate to **Identity** → **Users** → **All users** → **Alex Lee**.

4. In the left menu of Alex's profile, select **Overview** (or the main profile page).

5. Click **Revoke sessions** at the top of the page. Confirm when prompted.

6. You should see a confirmation notification: "Refresh tokens and authentication session cookies issued to Alex Lee have been revoked."

**Step 3: Observe Alex's current session**

7. Return to the InPrivate browser where Alex is signed in to the Office 365 portal.

8. Refresh the page. For apps with active access tokens (still within their 1-hour window), the page may still load normally — the access token in memory is still valid even though the refresh token was revoked.

9. Navigate to a different app — click on **SharePoint** or **Outlook** in the Office 365 portal.

10. When the current access token expires (or when the app attempts a silent refresh), Alex will be prompted to sign in again. In a CAE-enabled scenario, this re-authentication demand happens faster — within seconds to minutes.

**Step 4: Force re-authentication by navigating to a new resource**

11. In the InPrivate browser, navigate to `sharepoint.com` or directly to a SharePoint site.

12. If SharePoint is CAE-capable and Alex's sessions were revoked, SharePoint sends a challenge back to the browser requiring re-authentication. Alex is redirected to the sign-in page.

---

## Lab 3.6.2 — Configure Sign-In Frequency for SharePoint

**Tools:** Entra admin center
**Prerequisite:** Module 4 has not been started — this is an early CA preview

> 🔑 **Licensing:** Conditional Access policies require **Entra ID P1**. Your developer tenant includes this.

This is a preview of Conditional Access policy structure. Unit 4 covers CA policies in full depth. For now, follow the steps exactly.

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. Click **+ New policy**.

3. Name the policy: `SharePoint — Re-authentication every 8 hours`.

4. Under **Users**, click **0 users and groups selected**. Select **All users**. Click **Done**.

5. Under **Target resources**, click **No target resources selected**. Select **Cloud apps**. Click **Select apps**. Search for **Office 365 SharePoint Online** and select it. Click **Select** → **Done**.

6. Scroll to **Session** under **Access controls**. Click **0 controls selected**.

7. In the Session panel, check **Sign-in frequency**. Set the value to **8 Hours**. Uncheck **Persistent browser session** if shown. Click **Select**.

8. Under **Enable policy**, select **Report-only**. This allows the policy to run without enforcing it — so you can see what it would do in sign-in logs without affecting users.

9. Click **Create**.

10. Sign in as Alex Lee in an InPrivate browser. Navigate to SharePoint.

11. In the admin browser, find the sign-in in **Sign-in logs**. Open the sign-in detail. Navigate to the **Conditional Access** tab. Find the `SharePoint — Re-authentication every 8 hours` policy in the list. Its result should show **Report-only: Success** (would have applied, but not enforced).

**Find the Sign-In Frequency session detail:**

12. On the same sign-in log entry, look at the **Session info** tab or the **Session lifetime policies applied** field. It should show the sign-in frequency setting from the report-only policy.

---

## Lab 3.6.3 — Find the CAE Indicator in a Sign-In Log

**Tools:** Entra admin center
**Prerequisite:** Alex Lee has signed in to SharePoint or Exchange (from previous labs)

1. Navigate to **Identity** → **Monitoring & health** → **Sign-in logs**.

2. Filter by **User: Alex Lee** and find a recent sign-in to SharePoint Online or Exchange Online.

3. Click on the sign-in entry to open the detail pane.

4. Select the **Additional details** tab (or look for the CAE field in the Basic info tab depending on your portal version).

5. Look for a field labeled **Continuous access evaluation** or **CAE** in the details. In a supported scenario, this shows whether the sign-in was processed by a CAE-capable client and service combination.

6. If the field shows **Not applicable** or is absent: the client or service was not CAE-capable for this sign-in. Modern Office app sign-ins and recent Microsoft 365 client sign-ins typically show CAE as applicable.

> 💡 **Tip:** CAE coverage varies by client app version. Microsoft's own apps (Outlook for Windows, Teams, OneDrive sync client) consistently support CAE. Browser-based access through older versions of browsers or non-Microsoft clients may not show CAE as applicable.

---

## Scenario Check

Alex Lee's sessions were revoked. The immediate effect was visible in the next resource access attempt — SharePoint redirected Alex to re-authentication because SharePoint Online is a CAE-capable resource and received the revocation signal quickly.

The Sign-In Frequency policy for SharePoint is running in Report-Only mode. The sign-in log confirms the policy is being evaluated — and would require Alex to re-authenticate every 8 hours if enforced. In Unit 4, this policy will be switched to Enforce as part of the full Conditional Access stack.

The CAE indicator in the sign-in log shows that the sign-in between Alex's client and SharePoint Online was CAE-capable. This means future revocation signals will reach SharePoint within seconds to minutes — not waiting for the 1-hour access token expiry.

---

## Check Your Understanding

- [ ] From memory: state the default lifetime of an access token and a persistent refresh token
- [ ] Navigate to Alex Lee's profile and click **Revoke sessions** — confirm the success notification appears (this is a repeat from Lab 3.6.1 — do it from memory)
- [ ] Explain the gap between revoking sessions and actual access termination — how long is it without CAE?
- [ ] Navigate to a recent Alex Lee sign-in log entry and find the Conditional Access tab — confirm the SharePoint sign-in frequency policy appears as Report-only
- [ ] State two conditions required for CAE to function on a sign-in
- [ ] Explain the difference between Sign-In Frequency (a CA session control) and CAE

---

## Common Mistakes

**Assuming session revocation immediately terminates access.**
It terminates the ability to get new tokens. Existing valid tokens continue to work until they expire. For a compromised account, revoke sessions AND confirm CAE is active on the resources the attacker might be using. For immediate access termination, disabling the account forces re-authentication — but even then, CAE must be in the path for truly fast revocation.

**Setting Sign-In Frequency too aggressively.**
A 1-hour Sign-In Frequency on all apps means every user re-authenticates every hour across everything. This creates significant helpdesk load and user frustration, especially for users in time zones where their token expires mid-workday. Set Sign-In Frequency selectively — on high-sensitivity resources, not blanket across all apps.

**Confusing Sign-In Frequency with token lifetime policies.**
Token Lifetime policies (configurable via named policies on app registrations) and Sign-In Frequency (a CA session control) both affect how long users stay signed in. They interact. Sign-In Frequency overrides default token lifetimes for the CA policy scope. If both are configured for the same resource, the more restrictive applies. In most modern tenants, Token Lifetime policies are deprecated in favor of CA Sign-In Frequency.

**Not checking CAE coverage before claiming revocation is "immediate."**
CAE is not universal. If the user's client app does not support CAE, revocation is still eventually consistent at the token expiry boundary. Verify CAE coverage for the specific client and resource combination before claiming near-real-time revocation.

---

## What's Next

Module 3.7 is the Unit 3 scenario lab — you receive a brief describing Contoso's authentication security gaps, audit the current state, and fix all of it without step-by-step guidance. This pulls together everything from Modules 3.1 through 3.6: token behavior, authentication methods, MFA registration, SSPR, passwordless, and session management.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 3.5 — Passwordless Authentication](../3-5-passwordless-authentication/3-5-passwordless-authentication.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.7 — Secure Your Tenant's Authentication End to End →](../3-7-secure-tenant-authentication-end-to-end/3-7-secure-tenant-authentication-end-to-end.md)
