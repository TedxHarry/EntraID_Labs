# SAML SSO — Step by Step
*SAML is XML. When you decode it, you see exactly what Entra ID told the app about the user.*

> 🟡 **Intermediate** · Unit 5, Module 5.4 · ⏱️ 60 minutes

---

## What You Will Learn

- Explain what a SAML assertion is and what it contains
- Identify the three core SAML elements: Entity ID, ACS URL, and signing certificate
- Configure SAML SSO for an enterprise application in detail
- Decode an actual SAML assertion and read its claims
- Diagnose why SAML SSO fails and fix common errors

---

## Why This Matters

*SC-300: Implement access management for applications — Configure SAML-based single sign-on for applications*

Module 5.3 was practical: "Here's how to set up Slack." This module is technical: "Here's what SAML actually is and why you configure it that way." Enterprise SaaS (Salesforce, ServiceNow, Workday, Azure) all use SAML. When SSO fails silently — the most common problem — you need to understand SAML enough to decode the assertion and diagnose the problem. This module gives you that diagnostic skill.

---

## 📜 What Is a SAML Assertion

A SAML assertion is an XML document that Entra ID creates and sends to an app. It says: "I have verified this is Alex Lee from Contoso. Here is her email, name, groups, and job title. You can trust me because I signed this document with my certificate."

```xml
<saml:Assertion xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
    Version="2.0" ID="_a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    IssueInstant="2026-03-06T10:30:00Z">
    <saml:Issuer>https://sts.windows.net/12345678-1234-1234-1234-123456789012/</saml:Issuer>
    <saml:Subject>
        <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
            alex.lee@contoso.onmicrosoft.com
        </saml:NameID>
    </saml:Subject>
    <saml:Conditions NotBefore="2026-03-06T10:25:00Z"
        NotOnOrAfter="2026-03-06T10:35:00Z">
    </saml:Conditions>
    <saml:AttributeStatement>
        <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress">
            <saml:AttributeValue>alex.lee@contoso.com</saml:AttributeValue>
        </saml:Attribute>
        <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/displayname">
            <saml:AttributeValue>Alex Lee</saml:AttributeValue>
        </saml:Attribute>
    </saml:AttributeStatement>
    <saml:AuthnStatement AuthnInstant="2026-03-06T10:30:00Z">
    </saml:AuthnStatement>
</saml:Assertion>
```

**Key parts:**
- **Issuer:** Who created this (Entra ID)
- **Subject:** Who this is about (Alex Lee)
- **Conditions:** When this is valid (10 minutes, typically)
- **AttributeStatement:** User data (email, name, groups)
- **Signature:** Cryptographic proof Entra ID created it (not shown in XML, added separately)

---

## 🔐 The Three Things Slack Checks

When Slack receives this assertion, it does three things:

1. **Verify the signature** — "Did Entra ID really create this, or is it forged?"
2. **Check the timestamp** — "Is this assertion still valid (not expired)?"
3. **Read the claims** — "Who is this person, and what are their attributes?"

If any one of these fails, the user is not signed in. This is why SAML SSO fails silently — Slack silently rejects the assertion without telling the user why.

---

## ⚙️ The SAML Configuration Flow

**Step 1: You configure Entity ID and ACS URL (Contoso → Slack)**
- Entity ID: "I am Contoso at `https://sts.windows.net/<tenant-id>`"
- ACS URL: "Send assertions to `https://contoso.slack.com/sso/saml`"

**Step 2: Slack publishes its certificate (Slack → Contoso)**
- Slack gives Entra ID the certificate Entra ID should use to sign assertions
- Entra ID downloads it and stores it

**Step 3: User signs in at Slack**
- User clicks "Sign in with Contoso"
- Slack redirects to Entra ID

**Step 4: Entra ID authenticates and creates assertion (Entra ID)**
- User enters credentials
- Entra ID verifies them
- Entra ID creates a SAML assertion with the user's attributes
- Entra ID signs it with its certificate

**Step 5: Entra ID sends assertion to Slack's ACS URL**
- The assertion is sent as an HTTP POST to the ACS URL you configured

**Step 6: Slack verifies and grants access (Slack)**
- Slack verifies the signature
- Slack reads the email address from the assertion
- Slack finds or creates the user in its system
- Slack grants access

If any step fails, the user sees "authentication failed."

---

## 📋 Attribute Claims — Mapping Entra ID to Slack

The critical step is **attribute claims** — mapping what Entra ID calls something to what Slack calls it.

| Entra ID Attribute | Entra ID Claim Name | Slack Expects | Slack Maps To |
|---|---|---|---|
| email address | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` | `email` in SAML | Slack username |
| display name | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/displayname` | `name` in SAML | Slack display name |
| job title | `jobTitle` | `title` in SAML | Slack profile title |

If Entra ID sends `emailaddress` but Slack expects `email`, Slack won't find the user.

---

## Lab 5.4.1 — Configure SAML Attributes in Detail

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 5.3 complete; Slack SSO already set up

1. Navigate to **Applications** → **Enterprise applications** → **Slack** → **Single sign-on**.

2. Click **Edit** next to **User Attributes & Claims**.

3. You'll see the attribute mappings. The default Entra ID claims are:
   - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` → `user.mail`
   - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` → `user.givenname`
   - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` → `user.surname`
   - `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/displayname` → `user.displayname`

4. These map to Slack's expected fields. Do not change them unless Slack's documentation specifies different names.

5. For each attribute:
   - **Claim name** (left) — What Slack is looking for
   - **Value** (right) — Where Entra ID gets the data (e.g., `user.mail`)

6. Verify that `user.mail` is correct. If Contoso has a custom domain, make sure the email attribute maps to the right field.

7. Click **Save**.

---

## Lab 5.4.2 — Download and Decode the SAML Assertion

**Time:** 20 minutes
**Tools:** Entra admin center, browser developer tools
**Prerequisite:** Lab 5.4.1 complete, Slack SSO working

When a user signs in, the SAML assertion is sent as an HTTP POST in an HTML form. You can decode it to see exactly what Entra ID sent to Slack.

**Step 1: Capture the SAML assertion**

1. Open an **InPrivate** browser window.

2. Open **Developer Tools** (F12 or Right-click → Inspect).

3. Go to the **Network** tab.

4. Navigate to your Slack workspace: `https://<workspace>.slack.com`

5. Click "Sign in with Contoso" (or your SSO button).

6. You're redirected to Entra ID. Sign in with your credentials.

7. In the Network tab, look for a POST request to `contoso.slack.com/sso/saml` or similar (the ACS URL).

8. Click on that request. In the **Request** tab, look for the POST body.

9. You'll see a parameter called `SAMLResponse`. This is the SAML assertion (base64-encoded).

10. Copy the `SAMLResponse` value (it's a long base64 string).

**Step 2: Decode the assertion**

11. Open a new tab and go to `https://samltool.io` or `https://www.samltool.com/decode.php`

12. Paste the `SAMLResponse` value into the decoder.

13. Click **Decode**. The XML SAML assertion is now visible.

14. Read the decoded assertion:
    - **Issuer** — Should be your Entra ID tenant (sts.windows.net/<tenant-id>)
    - **Subject** — Should be the user's email (alex.lee@contoso.onmicrosoft.com)
    - **AttributeStatement** — Should include email, displayname, givenname, surname
    - **Conditions** — Check the NotOnOrAfter timestamp (is it in the future?)

---

## Lab 5.4.3 — Diagnose a SAML Failure

**Time:** 15 minutes
**Tools:** Entra admin center, browser developer tools
**Prerequisite:** Lab 5.4.2 complete

If SSO fails, the assertion is usually the culprit. You'll now learn to diagnose the most common failure.

**Scenario:** Morgan Chen signs in but gets "SAML assertion validation failed."

**Step 1: Check the assertion timestamp**

1. Capture and decode the SAML assertion for Morgan (repeat Lab 5.4.2 steps).

2. Look at the `<saml:Conditions>` element. Check:
   - `NotBefore` — Assertion is valid after this time
   - `NotOnOrAfter` — Assertion expires at this time
   - Typically: 5 minutes before now to 10 minutes after now

3. If `NotOnOrAfter` is in the past, the assertion has expired. This usually means:
   - Entra ID's clock is not synced with Slack's clock (rare)
   - The user took too long to complete sign-in (sign-in flow timed out)

4. If the timestamp is fine, move to Step 2.

**Step 2: Check the attribute mapping**

5. In the decoded assertion, find the `<saml:AttributeStatement>` section.

6. Look for the `email` attribute. Does it match what Slack expects?
   - Slack usually expects the primary email (e.g., `morgan.chen@contoso.com`)
   - If the attribute shows a secondary email or is missing, mapping failed

7. If the email attribute is missing entirely, check the attribute claims in Entra ID (Lab 5.4.1). Is `user.mail` mapped correctly?

**Step 3: Check the issuer and subject**

8. In the assertion, verify:
   - **Issuer** — Must match what Slack expects (usually `https://sts.windows.net/<tenant-id>/`)
   - **Subject/NameID** — Must match the user's Entra ID identifier (email or UPN)

9. If the issuer is wrong, the Entity ID configuration in Entra ID is incorrect.

**Common fixes:**
- Wrong attribute mapping → Fix in "User Attributes & Claims"
- Wrong Entity ID → Fix in "Basic SAML Configuration"
- Wrong ACS URL → Fix in "Basic SAML Configuration"
- Missing user attributes in Entra ID → Add missing properties to the user profile

---

## The Scenario Check

Contoso's Slack SAML SSO is now fully configured and diagnostic. When Alex Lee signs in to Slack, Entra ID creates a SAML assertion with her email (`alex.lee@contoso.com`), display name (`Alex Lee`), and other attributes. Entra ID signs the assertion with its certificate, sends it to Slack's ACS URL, and Slack verifies the signature and grants access.

When Morgan Chen signs in and gets an error, Contoso's admin (Jordan Kim) can now:
1. Capture the SAML assertion using browser developer tools
2. Decode it on a SAML decoder tool
3. Check the timestamp, issuer, and attributes
4. Identify whether the problem is a mapping issue, configuration issue, or user profile issue
5. Fix it without guessing

This diagnostic skill prevents the common mistake: "SAML is broken, let me call Salesforce support" when the issue is a simple attribute mapping.

---

## Check Your Understanding

- [ ] Navigate to **Slack** → **Single sign-on** → **User Attributes & Claims**. Identify which Entra ID attribute maps to which claim name.
- [ ] Sign in to Slack and capture the SAML assertion using browser developer tools. Decode it on samltool.io.
- [ ] In the decoded assertion, find the Issuer, Subject, and at least three attributes.
- [ ] Explain why the timestamp in a SAML assertion matters.
- [ ] Describe what happens if the ACS URL in "Basic SAML Configuration" is wrong.

---

## Common Mistakes

**Assuming SAML claims must match the user's display name exactly.**
You change Alex's display name from "Alex Lee" to "Alexandra Lee" in Entra ID. The next SAML assertion includes "Alexandra Lee," but Slack already has the user as "Alex Lee." The names don't match, and Slack creates a duplicate account. SAML claims must be *stable and unique* — usually email addresses.

**Not checking timestamp validity when debugging.**
SSO fails. You decode the SAML assertion and see all the claims are correct. You assume the problem is permissions or groups. You forget to check the `NotOnOrAfter` timestamp. The assertion expired because the sign-in flow took too long. Always check timestamp first.

**Modifying claims without understanding what the app expects.**
Slack documentation doesn't say which claims it needs. You assume it needs `groups` and add a group claim to the assertion. The assertion now includes `groups: [Finance]`. Slack doesn't use groups in SAML, so it's ignored. You've added unnecessary complexity. Only add claims that the documentation specifies.

**Using display name or UPN as the unique identifier.**
You configure the Subject/NameID to use display name. Alex Lee changes her name to Alex Johnson. The next SAML assertion shows "Alex Johnson," but Slack has the user as "Alex Lee." The user can't sign in. Always use email address (immutable) as the identifier, not display name.

---

## What's Next

Module 5.5 teaches **OIDC/OAuth SSO** — the modern alternative to SAML. OIDC is simpler, uses JSON instead of XML, and is the standard for modern applications. SAML is enterprise-legacy (older apps, Salesforce, ServiceNow). OIDC is the future. You'll configure both and understand when to use each.

---

> 📚 **Entra ID — Zero to Admin** · Unit 5: Applications and SSO
>
> [← Module 5.3 — Adding a Gallery App and First SSO](../5-3-adding-gallery-app-sso/5-3-adding-gallery-app-sso.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 5.5 — OIDC/OAuth SSO →](../5-5-oidc-oauth-sso/5-5-oidc-oauth-sso.md)
