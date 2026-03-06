# From Password to Token: What Authentication Actually Produces
*Your password gets you in. A token is what you carry once you're in — and it's not a secret.*

> 🟢 **Beginner** · Unit 3, Module 3.1

---

## What You Will Learn

- State the three token types Entra ID issues and what each one is for
- Identify the six key claims inside an access token and explain what each one means
- Decode a real access token at jwt.ms and read its claims
- See how a profile change in Entra ID appears in a fresh token (and why old tokens still carry the old value)
- Call the Microsoft Graph `/me` endpoint and understand what the token claims are authorizing

---

## Why This Matters

*SC-300: Implement authentication and access management — Understand authentication concepts in Microsoft Entra ID*

Module 1.4 traced the sign-in flow at a high level — five steps from typing a password to getting access. This module goes one level deeper into what Entra ID actually hands back when authentication succeeds: a set of cryptographically signed tokens. Every application that uses Microsoft identity relies on these tokens. Every Conditional Access decision you build in Unit 4 results in these tokens being issued or denied. Every security investigation in Unit 6 starts by reading token claims in sign-in logs. Understanding tokens is not optional — it is the foundation that everything else in this series builds on.

---

## 🎟️ Three Tokens, Three Jobs

When a user signs in successfully, Entra ID does not simply say "yes, allowed." It issues tokens — cryptographically signed JSON objects that prove who the user is and what they're allowed to do.

| Token | What it is | What it's for | Lifetime |
|---|---|---|---|
| **ID token** | Proof of who the user is | Tells the app the user's identity (name, email, Object ID) | Short — typically 1 hour |
| **Access token** | Proof of what the user can do | Presented to an API (like Microsoft Graph) to request data | Short — typically 1 hour |
| **Refresh token** | A ticket to get new tokens | Exchanged silently for new access and ID tokens when they expire | Long — typically 90 days |

The separation matters. The access token is what Microsoft Graph, SharePoint, Teams, and any API reads to decide what the user is allowed to do. The ID token is what the application reads to personalize the UI ("Hello, Alex Lee"). The refresh token stays in the browser or the app — the user doesn't see it, but it's why they don't have to sign in again every hour.

> 🔑 **Key concept:** An access token is not a password. It is a signed statement from Entra ID saying "at this time, this user is allowed to do these things." Any application that receives the access token can read those claims without calling Entra ID again — it just verifies the signature.

---

## 🔍 What Lives Inside a Token

Tokens are JSON Web Tokens (JWTs). A JWT has three parts, separated by dots:

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20v...eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
```

- Part 1 (header): algorithm and token type
- Part 2 (payload): the claims — the actual data
- Part 3 (signature): cryptographic proof the token was issued by Entra ID

Only the payload matters for your purposes. Every useful piece of information lives there.

**Key claims in an Entra ID access token:**

| Claim | Name | What it contains | Example |
|---|---|---|---|
| `iss` | Issuer | Which Entra ID tenant issued this token | `https://login.microsoftonline.com/{tenant-id}/v2.0` |
| `sub` | Subject | A unique identifier for the user (per-app) | Long GUID — changes per application |
| `oid` | Object ID | The user's Object ID in Entra ID — consistent across all apps | Same GUID as shown on the user's profile |
| `tid` | Tenant ID | The tenant this token was issued from | Your tenant's GUID |
| `aud` | Audience | Which API this token is intended for | `https://graph.microsoft.com` for Graph tokens |
| `scp` | Scope | What the user has authorized the app to do | `User.Read profile openid email` |
| `iat` | Issued at | Unix timestamp when the token was created | `1706800000` |
| `exp` | Expiry | Unix timestamp when the token expires | `1706803600` (1 hour after iat) |
| `name` | Display name | The user's display name at time of token issuance | `Alex Lee` |
| `upn` | UPN | The user's sign-in name | `alex.lee@yourdomain.onmicrosoft.com` |

The `oid` claim is the most important for identity. It is the same Object ID you see in the Entra admin center on Alex's profile. It never changes, even if Alex's name or email changes. APIs use `oid` as the primary user identifier — not the display name, not the UPN.

---

## ⏱️ Why Old Tokens Can Have Stale Claims

A critical behavior: **tokens are not updated in real time.**

When Entra ID issues an access token, the claims (name, job title, department) are captured at that moment and baked into the signed token. The token has a 1-hour lifetime. During that hour, if you change Alex's job title from "Finance Analyst" to "Senior Finance Analyst" in the Entra admin center, any app that checks Alex's job title from the existing token still sees the old value.

The new value appears in the next token — issued when the current one expires or when the app explicitly requests a new one.

This behavior is intentional. It reduces the number of calls to Entra ID by allowing apps to trust the token for its lifetime. The trade-off: profile changes take up to one token lifetime to propagate to all apps. For display name or job title changes, this is fine. For security-critical changes (account disabled, password changed), it is not — this is where Continuous Access Evaluation (CAE) in Module 3.6 becomes important.

---

## Lab 3.1.1 — Decode an Access Token at jwt.ms

**Tools:** Graph Explorer (`aka.ms/ge`), jwt.ms
**Prerequisite:** Alex Lee exists from Module 2.1. You are signed in as your Global Administrator.

**Step 1: Get Alex Lee's access token using Graph Explorer**

1. Open a new InPrivate/private browser window.

2. Navigate to `aka.ms/ge` (Graph Explorer). Sign in as **Alex Lee** using Alex's UPN and the password `Contoso@1234!` (set in Module 2.1).

3. Graph Explorer opens showing Alex's identity in the top right. The default query is:
   ```
   GET https://graph.microsoft.com/v1.0/me
   ```

4. Click **Run query**. The response appears in the bottom panel — Alex's profile data returned by the `/me` endpoint.

5. To get the actual access token Graph Explorer is using, click the **Access token** tab at the top of the request panel (between **Modify permissions** and **Request headers**).

6. The access token appears as a long string starting with `eyJ`. Copy the entire token to your clipboard.

**Step 2: Decode the token at jwt.ms**

7. Open a new tab and navigate to `jwt.ms`.

8. Paste the copied access token into the text box. The page automatically decodes it and displays the claims.

9. Find and record these values from the decoded token:
   - `iss` — note the tenant GUID embedded in the URL
   - `oid` — Alex Lee's Object ID
   - `tid` — your tenant ID
   - `aud` — what API this token is for
   - `scp` — what permissions were granted
   - `exp` — the expiry timestamp (convert from Unix time if curious: `exp` is seconds since Jan 1, 1970)

10. Navigate to the Entra admin center in your regular browser (not InPrivate). Go to **Identity** → **Users** → **All users** → **Alex Lee** → **Overview**.

11. Compare the **Object ID** shown on Alex's profile in the admin center against the `oid` claim in the decoded token. They must match.

---

## Lab 3.1.2 — Change a Profile Attribute and Watch the Token Refresh

**Tools:** Entra admin center, Graph Explorer, jwt.ms
**Prerequisite:** Lab 3.1.1 complete. Alex's InPrivate Graph Explorer session is still open.

1. In your regular browser (admin session), navigate to **Alex Lee** → **Properties** → **Job info**. Change Alex's **Job title** from `Finance Analyst` to `Senior Finance Analyst`. Click **Save**.

2. Return to Alex's InPrivate Graph Explorer tab.

3. Click **Run query** again on the GET `/me` endpoint. Look at the `jobTitle` field in the response. Note whether it shows the old or new value.

4. The response from `/me` often shows near-real-time data pulled from the directory — not from the token. The token itself may still carry the old value.

5. To check the token claims specifically: click the **Access token** tab again. Copy the token. Paste it into jwt.ms. Look for the `jobTitle` claim in the token payload.

6. If the old session token is still valid (less than 1 hour has passed since sign-in), the `jobTitle` in the token still shows the old value even though the directory was just updated.

7. To force a new token: in the InPrivate window, sign out of Graph Explorer (click Alex's avatar in the top right → **Sign out**). Sign back in as Alex.

8. After re-authentication, copy the new access token from the **Access token** tab and decode it at jwt.ms. The `jobTitle` claim in the new token should now show `Senior Finance Analyst`.

9. This confirms the behavior: the token carries a point-in-time snapshot of claims at issuance. Updates to the user profile appear in the next token — not in the current one.

---

## Lab 3.1.3 — Read What the Token Authorizes via the /me Endpoint

**Tools:** Graph Explorer
**Prerequisite:** Alex Lee signed in to Graph Explorer

1. In Alex's Graph Explorer session, ensure the GET `/me` query is selected.

2. Click **Run query**. The full response shows Alex's profile as Microsoft Graph returns it from the token claims combined with the directory.

3. The response includes these fields that come from token claims:
   - `id` — maps to the `oid` claim
   - `displayName` — maps to the `name` claim
   - `userPrincipalName` — maps to the `upn` claim
   - `mail` — Alex's email address

4. Now try to access someone else's data. Change the request URL to:
   ```
   https://graph.microsoft.com/v1.0/users
   ```

5. Click **Run query**. If Alex's token only has `User.Read` scope (the default for the `/me` permission), this request will fail with a 403 Forbidden error. The `scp` claim in Alex's token does not include `User.Read.All` — so the API denies the request.

6. This demonstrates the audience and scope claims working as designed: Alex's token is for `graph.microsoft.com`, and it only authorizes the specific operations listed in the `scp` claim.

---

## Scenario Check

Alex Lee signed in to Graph Explorer, and the token Entra ID issued was decoded and read in full. The `oid` claim in Alex's token matched the Object ID on Alex's profile in the admin center — confirming that the token is tied to Alex's specific identity object in the directory.

When Alex's job title was updated from Finance Analyst to Senior Finance Analyst, the existing access token still showed the old value. Only after re-authentication did a fresh token carry the updated claim. This is the expected behavior for any non-security-critical profile change — the update propagates on the next token refresh.

The scope limitation was visible when Alex's token was used to query all users — the `scp` claim did not include `User.Read.All`, so Graph rejected the request. The token is not a master key — it is scoped exactly to what the user authorized.

---

## Check Your Understanding

- [ ] Navigate to jwt.ms — paste a fresh access token from Graph Explorer (signed in as Alex) and find the `oid`, `tid`, and `scp` claims
- [ ] From memory: state the difference between an ID token and an access token
- [ ] From memory: state why a profile change in Entra ID does not immediately appear in an existing access token
- [ ] Explain what happens when an API receives an access token with `aud` set to a different service than the one being called
- [ ] State which claim in the token would you use as a primary user identifier in a database — and why not the display name
- [ ] What is the typical lifetime of an access token, and what is the typical lifetime of a refresh token?

---

## Common Mistakes

**Treating the access token as a secret.**
Access tokens are signed but not encrypted. Anyone who has the token can read its claims. This is by design — applications need to read the claims to make authorization decisions. The security guarantee is the signature — the token was issued by Entra ID and has not been tampered with. Treat tokens as sensitive (don't log them, don't put them in URLs), but understand that their contents are readable.

**Assuming a profile change takes effect immediately in all apps.**
An admin changes a user's department. The user continues working. For up to 1 hour, every access token the user has contains the old department value. Any app that reads the department from the token will see the old value. This is the expected behavior. If immediate propagation is required (security policy, compliance), session revocation (Module 3.6) and CAE are the tools.

**Confusing `sub` with `oid`.**
Both are GUIDs. `sub` is per-application — the same user will have a different `sub` value in different apps. `oid` is consistent across all apps and matches the Object ID in the Entra admin center. Always use `oid` when you need a stable user identifier.

**Using the `/me` endpoint to check what another user can see.**
The `/me` endpoint returns data for the signed-in user — always. To read another user's data, you need a different endpoint and a token with broader scopes. The scope on a user's personal token is intentionally limited to their own data.

---

## What's Next

Module 3.2 covers the Authentication Methods Policy — the tenant-wide configuration that controls which authentication methods (password, Microsoft Authenticator, FIDO2 keys, SMS) users can register and use. The tokens you decoded in this module are only issued after the user successfully authenticates. Module 3.2 is about controlling what that authentication step looks like.

---

> 📚 **Entra ID — Zero to Admin** · Unit 3: Authentication
>
> [← Module 2.9 — Administrative Units](../../UNIT 2 - USERS, GROUPS AND EXTERNAL IDENTITIES/2-9-administrative-units/2-9-administrative-units.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 3.2 — The Authentication Methods Policy →](../3-2-authentication-methods-policy/3-2-authentication-methods-policy.md)
