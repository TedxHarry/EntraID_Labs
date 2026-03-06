# How Entra ID Makes Decisions
*You can trace any sign-in from start to finish, read every field in a log entry, and decode an access token.*

> 🟢 **Beginner** · Unit 1, Module 1.4

---

## What You Will Learn

- Describe the five steps Entra ID goes through on every sign-in attempt
- Identify what can fail at each step and what the failure looks like in the logs
- Read a sign-in log entry and explain every field shown in the detail pane
- Decode an access token at `jwt.ms` and identify the key claims it contains
- Locate a failed sign-in in the logs by its error code and failure reason

## Why This Matters

Every policy in this series — MFA, Conditional Access, Identity Protection, PIM — operates at a specific step in the sign-in flow. If you do not know the flow, you cannot predict what a policy does, debug why access was denied, or explain to a user why they are being blocked. This mental model is the foundation. Every module in Units 3 through 9 builds directly on it.

---

## 🚦 Five Steps Between "Enter Password" and "Access Granted"

Every time a user signs in — to Outlook, SharePoint, the Azure portal, any connected app — Entra ID executes the same five steps in sequence. If any step fails, the sign-in stops there. The steps after a failure do not run.

### Step 1 — User Realm Discovery

The user types their email address. Entra ID looks up the domain portion (`@contoso.com`) to determine how this identity is managed:

- **Managed (cloud-only):** Entra ID handles authentication directly.
- **Managed (synced):** Entra ID handles authentication using a hash of the on-prem password (Password Hash Sync).
- **Federated:** Authentication is delegated to an external identity provider (ADFS, Okta, Ping). The user is redirected there.

What can fail here: the user does not exist in any tenant, or the domain is not registered in Entra ID.

### Step 2 — Authentication

The user proves who they are. For cloud-only and synced accounts, Entra ID validates the submitted credential (password, FIDO2 key, Windows Hello, Authenticator app) directly. For federated accounts, the external IdP handles this step.

If MFA is required — either by the Authentication Methods Policy or by a Conditional Access policy — the user receives the MFA challenge at this point.

What can fail here: wrong password, MFA challenge not satisfied, account locked after too many failed attempts, account disabled.

### Step 3 — Conditional Access Evaluation

Authentication succeeded. The user proved who they are. Now Entra ID evaluates every Conditional Access policy that applies to this sign-in.

The evaluation considers: who is signing in, which app they are accessing, which device they are on, where they are signing in from, and the current risk level of the sign-in.

Every applicable policy produces a result. The final outcome is the most restrictive combination of all results. One Block policy overrides all Grant policies.

What can fail here: a CA policy blocks the sign-in, or a CA policy requires something the user cannot satisfy (no compliant device, no MFA registered, signing in from a blocked country).

### Step 4 — Token Issuance

All checks passed. Entra ID generates three tokens and sends them to the application:

| Token | Purpose | Lifetime (default) |
|---|---|---|
| **ID token** | Tells the app who the user is (identity claims) | Short-lived |
| **Access token** | Authorizes the app to call APIs on behalf of the user | 60–90 minutes |
| **Refresh token** | Used to get new access tokens without prompting the user again | Up to 90 days |

What can fail here: token issuance errors are rare but can occur if there is a misconfigured app registration or an incompatible token version.

### Step 5 — Access Granted

The app receives the token and validates it: checks the cryptographic signature, confirms the audience matches the app, confirms the token has not expired. If valid, the app uses the claims inside the token to determine what the user can do.

This step happens inside the application, not in Entra ID. The Entra sign-in log does not capture what happens inside the app after the token is delivered.

> 🔑 **Key concept:** Authentication (Step 2) and authorization (Step 3) are separate. A user can successfully authenticate — prove who they are — and still be blocked by Conditional Access before they receive a token. The sign-in log records both outcomes separately.

---

## 📋 Every Sign-In Is Recorded. Here Is How to Read It.

Every sign-in attempt — successful or failed — produces a log entry in **Identity → Users → Sign-in logs**. Log entries are retained for 30 days (Entra ID Free) or up to 90 days (Entra ID P1/P2).

Clicking any log entry opens a detail pane with five tabs: **Basic info**, **Location**, **Device info**, **Authentication**, and **Conditional Access**.

**The fields you will use most:**

| Field | Where it appears | What it tells you |
|---|---|---|
| **Date** | Row in list | When the sign-in attempt occurred |
| **User** | Row and Basic info | Who attempted to sign in (display name + UPN) |
| **Application** | Row and Basic info | Which app they were signing in to |
| **Status** | Row | Success, Failure, or Interrupted |
| **Sign-in error code** | Basic info (on failure) | Numeric code — look this up to get the exact reason |
| **Failure reason** | Basic info (on failure) | Plain-text description of why sign-in failed |
| **IP address** | Location tab | The IP the sign-in came from |
| **Location** | Location tab | City and country inferred from the IP |
| **Client app** | Basic info | Browser, mobile app, legacy auth client, or Exchange ActiveSync |
| **Authentication requirement** | Authentication tab | Single-factor or multifactor |
| **Authentication method** | Authentication tab | Password, Microsoft Authenticator, FIDO2, etc. |
| **MFA result** | Authentication tab | Whether MFA was satisfied, skipped, or failed |
| **Conditional Access** | Conditional Access tab | Success, Failure, or Not Applied — with each policy listed |

**The most useful field when something goes wrong: Sign-in error code.**
Look up any code at `learn.microsoft.com/entra/identity-platform/reference-error-codes`. The two you will see most often in this series:

| Error code | Meaning |
|---|---|
| `50126` | Invalid username or password |
| `50057` | User account is disabled |
| `53003` | Access blocked by Conditional Access |
| `50076` | MFA required (this is a step, not a final failure) |
| `700016` | Application not found in tenant |

---

## 🎫 The Access Token Is Not a Secret

Most people assume the access token must be protected because it grants access. The token does need to be protected from theft — but not because it is encrypted. The token is readable by anyone who has it. Open one at `jwt.ms` and you can read exactly what is inside.

The security comes from the **signature**. Entra ID signs every token with its private key. Any app can verify the signature using Entra ID's public key (published at a well-known URL). A forged or modified token produces an invalid signature and is rejected.

A decoded access token contains claims — key-value pairs that describe the user, the token's purpose, and its validity window.

**Key claims in an access token:**

| Claim | Name | What it contains |
|---|---|---|
| `iss` | Issuer | The Entra ID endpoint that issued the token |
| `sub` | Subject | A unique identifier for the user in this specific app |
| `oid` | Object ID | The user's Entra ID Object ID — consistent across all apps |
| `tid` | Tenant ID | The GUID of the tenant that issued the token |
| `aud` | Audience | Which app or API this token is for |
| `upn` | User Principal Name | The user's UPN (e.g., `alex@contoso.com`) |
| `name` | Display Name | The user's display name |
| `scp` | Scope | What permissions the app has on behalf of the user |
| `iat` | Issued At | Unix timestamp when the token was issued |
| `exp` | Expiry | Unix timestamp when the token expires |
| `nbf` | Not Before | Unix timestamp before which the token is not valid |

> 🔑 **Key concept:** `sub` and `oid` are both user identifiers, but they serve different purposes. `oid` is the same across every app in the same tenant — it is the user's Entra ID Object ID. `sub` is unique per app and per user — two different apps see different `sub` values for the same user. Use `oid` to identify a user consistently across services.

---

## Lab 1.4.1 — Trace a Sign-In Through the Logs

**Tools:** Entra admin center, InPrivate browser window
**Prerequisite:** Test User 01 created in Module 1.1. Know their UPN and password.

1. Open a new **InPrivate** or **Incognito** browser window. Go to `myapps.microsoft.com`.

2. Sign in as Test User 01 (`testuser01@yourdomain.onmicrosoft.com`) with the correct password. Complete the sign-in successfully. You should reach the My Apps portal. Close the InPrivate window.

3. In your admin browser, navigate to **Identity → Users → Sign-in logs**.

4. Find the sign-in for Test User 01 — it should appear within 1–2 minutes. Click **Refresh** if needed. Click the row to open the detail pane.

5. On the **Basic info** tab, read and note down:
   - **Date** — the exact timestamp
   - **Request ID** — a unique identifier for this specific sign-in attempt
   - **Correlation ID** — used to link related requests together
   - **User** — confirm it shows Test User 01's display name and UPN
   - **Application** — which app was accessed
   - **Status** — confirm it shows **Success**
   - **Client app** — what browser/client was used

6. Click the **Location** tab. Note:
   - **IP address** — your home or work IP
   - **City** and **Country** — may show approximate or incorrect location (IP geolocation is not precise)

7. Click the **Authentication** tab. Note:
   - **Authentication requirement** — does it show Single-factor or Multifactor?
   - **Authentication method** — should show **Password**
   - **Authentication step** — look for "Primary authentication" in the list

8. Click the **Conditional Access** tab. Note:
   - **Conditional Access result** — should show **Not applied** (no policies exist yet)

**You'll know this worked when** you can identify the Status, Application, IP address, Authentication method, and Conditional Access result for the sign-in.

---

## Lab 1.4.2 — Decode an Access Token at jwt.ms

**Tools:** `jwt.ms`, Graph Explorer (`aka.ms/ge`), InPrivate browser window
**Prerequisite:** Lab 1.4.1 completed.

**Part A — Decode your ID token at jwt.ms**

1. Open a new **InPrivate** or **Incognito** browser window. Go to `jwt.ms`.

2. The jwt.ms page has a **Sign in** button. Click it. Sign in with your **sandbox admin account** (not Test User 01 — use your main admin account so you can see richer claims).

3. After signing in, jwt.ms displays the decoded ID token in the left pane and the raw token in the right pane. Do not close this window.

4. In the decoded token, find and note the value of each claim:
   - `iss` — what Entra ID endpoint issued this token? Note the tenant ID embedded in the URL.
   - `oid` — copy this value. This is your admin account's Object ID. Verify it: in your admin browser, navigate to **Identity → Users → All users**, open your admin account, and compare the **Object ID** shown on the profile page.
   - `tid` — compare this to the Tenant ID you found in Module 1.2. They should match exactly.
   - `name` — your display name
   - `preferred_username` — your UPN
   - `exp` — this is a Unix timestamp. Go to `unixtimestamp.com` and convert it — how long is this token valid?

5. Scroll down in the decoded token. Find the `aud` claim. For an ID token issued by jwt.ms, the audience will be the jwt.ms application ID. This confirms the token is scoped to jwt.ms and cannot be used to call another API.

**Part B — Get an access token from Graph Explorer**

6. In a new tab, go to `aka.ms/ge` (Graph Explorer). Sign in with your sandbox admin account when prompted.

7. In Graph Explorer, click the **user icon** (your initials) in the top-right corner. In the dropdown, click **Access token**. A dialog appears showing the raw access token — a long string of characters.

8. Copy the entire access token string. It starts with `eyJ`.

9. Go back to the `jwt.ms` tab. In the **Token** input box at the top right, paste the access token and click **Decode**. The decoded claims appear.

10. In the decoded access token, find and compare:
    - `aud` — for a Graph Explorer token, the audience is `https://graph.microsoft.com`. This token is scoped to the Microsoft Graph API — it cannot be used to call a different API.
    - `scp` — the permissions (scopes) this token has. You will see the permissions Graph Explorer was granted.
    - `oid` — confirm it matches the Object ID you found in Part A. Same user, same `oid`, across both tokens.
    - `exp` — when does this access token expire? Compare to the ID token expiry. Access tokens are shorter-lived.

**You'll know this worked when** you can explain the difference between the `aud` claim in the ID token vs the access token, and you can confirm your `oid` matches your user's Object ID in the admin center.

---

## Lab 1.4.3 — Simulate Failures and Find Them in the Logs

**Tools:** Entra admin center, InPrivate browser window
**Prerequisite:** Labs 1.4.1 and 1.4.2 completed.

**Failure 1 — Wrong password**

1. Open a new **InPrivate** browser window. Go to `myapps.microsoft.com`.

2. Sign in as Test User 01 but enter an intentionally wrong password. You should see a sign-in error message. Close the InPrivate window.

3. In your admin browser, navigate to **Identity → Users → Sign-in logs**.

4. Find the failed sign-in for Test User 01. It will have a **red dot** or **Failure** status. Click it.

5. On the **Basic info** tab, locate:
   - **Status** — Failure
   - **Sign-in error code** — note the number
   - **Failure reason** — read the plain-text description

   The error code for a wrong password is `50126`. The failure reason reads: *"Error validating credentials due to invalid username or password."*

6. Compare this log entry to the successful sign-in from Lab 1.4.1. What is different? What is the same?

**Failure 2 — Disabled account**

7. In your admin browser, navigate to **Identity → Users → All users**. Click on **Test User 01**.

8. On the user's profile page, under the **Properties** tab, find **Account enabled**. Click **Edit** (or use the toggle directly if shown). Set **Account enabled** to **No**. Click **Save**.

   You should see a confirmation that the user was updated. Test User 01's account is now disabled.

9. Open a new **InPrivate** browser window. Go to `myapps.microsoft.com`. Sign in as Test User 01 with the correct password. You will see an error message stating the account is disabled.

10. In your admin browser, navigate to **Sign-in logs**. Find this attempt. Click it and read:
    - **Status** — Failure
    - **Sign-in error code** — `50057`
    - **Failure reason** — *"The user account is disabled."*

11. Note where the sign-in stopped. Entra ID did not even reach the Conditional Access evaluation step — the account was blocked at Step 2 (Authentication). This is visible because the Conditional Access tab shows **Not Applied**, not **Failure**.

12. Re-enable Test User 01. Navigate back to their profile, set **Account enabled** to **Yes**, and click **Save**.

**You'll know this worked when** you can locate both failed sign-ins in the logs, state the error code for each, and explain at which step in the five-step flow each failure occurred.

---

## The Scenario Check

The five-step sign-in flow is now mapped and observed in the logs. Test User 01's sign-in was traced from password submission through token issuance. Two failures were induced and located in the sign-in logs: a wrong password (error 50126, fails at Step 2) and a disabled account (error 50057, fails at Step 2). The access token was decoded and the key claims were read. No Contoso characters exist in the tenant yet — they are created in Module 2.1. Every lab from Unit 2 onward produces sign-in log entries that follow exactly this five-step flow.

---

## Check Your Understanding

- [ ] Without reading this module, name the five steps Entra ID executes on every sign-in in order.
- [ ] A user signs in successfully but still cannot access an app. At which step did the block occur? How would the Conditional Access tab in the sign-in log look?
- [ ] Navigate to **Sign-in logs** and filter by **Status: Failure**. Find any failure and state its error code and failure reason.
- [ ] Open `jwt.ms` and sign in. Find the `oid` claim. Verify it matches the Object ID shown on your user profile in the admin center.
- [ ] State the difference between the `oid` claim and the `sub` claim in an access token.
- [ ] State what the `aud` claim in an access token controls and why it matters for security.

---

## Common Mistakes

**Assuming a successful sign-in means the user got into the app.**
Entra ID's job ends when the token is issued. What happens inside the app after that — role checks, license checks, account provisioning — is not captured in the Entra sign-in log. If a user signs in successfully in the log but reports they cannot access the app, the problem is in the app's authorization layer, not in Entra ID.

**Reading only the Status column and not the error code.**
"Failure" in the Status column tells you something went wrong. The **Sign-in error code** tells you exactly what went wrong. Always click into the log entry and read the error code and failure reason. Looking up the code at `learn.microsoft.com/entra/identity-platform/reference-error-codes` gives the exact cause and resolution steps.

**Confusing the ID token and the access token.**
The ID token tells the application who the user is. The access token grants the application permission to call APIs. They look similar when decoded (both are JWTs) but they have different audiences (`aud`), different scopes, and different lifetimes. Using an ID token where an access token is required will fail at the API.

---

## What's Next

Unit 2 begins with Module 2.1 — Creating and Managing Users. You will create all five Contoso characters (Alex Lee, Morgan Chen, Jordan Kim, Sam Taylor, River Patel) with correct properties, and do it two ways: through the admin center portal and through Microsoft Graph PowerShell. Every policy in Units 3 through 13 targets these users. Set them up correctly in Module 2.1 and every subsequent lab works. Skip a property or get a detail wrong and a lab in Unit 5 or Unit 7 will fail for a reason that is hard to trace back.

---

> 📚 **Entra ID — Zero to Admin** · Unit 1: Getting Started
>
> [← Module 1.3 — The Entra Admin Center](../1-3-entra-admin-center/1-3-entra-admin-center.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | *End of Unit 1 — [Continue to Unit 2: Users, Groups and External Identities →](../../UNIT%202%20-%20USERS%2C%20GROUPS%20AND%20EXTERNAL%20IDENTITIES/2-1-creating-managing-users/2-1-creating-managing-users.md)*
