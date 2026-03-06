# Named Locations — Trusted and Untrusted Networks
*Your office network is not the same as an unknown hotel Wi-Fi. Conditional Access can tell the difference.*

> 🟢 **Beginner** · Unit 4, Module 4.4

---

## What You Will Learn

- Create an IP-based Named Location for a trusted network and mark it as trusted
- Create a country-based Named Location to define allowed sign-in countries
- Use a Named Location as a CA condition to require stricter controls from untrusted locations
- Explain the difference between trusted and non-trusted locations in CA evaluation

---

## Why This Matters

*SC-300: Implement authentication and access management — Configure and manage named locations*

Location is one of the strongest contextual signals available to Conditional Access. A Finance Analyst signing in from the office at 9am on Monday is expected. The same analyst signing in from a foreign country at 3am is suspicious. Named Locations let you codify "trusted" and use that context in your policies. You can require MFA from everywhere, or you can require MFA only from untrusted locations. You can block sign-ins from countries Contoso has no business in. Location conditions are simple to configure and significantly raise the cost of account compromise.

---

## 📍 Two Types of Named Locations

**IP-range locations:** Define a set of IP addresses or CIDR ranges. Typically used for office networks, VPN exit nodes, and known partner networks. Can be marked as **trusted** — sign-ins from trusted IP ranges receive a slightly higher implicit trust signal in CA evaluation and in Identity Protection risk scoring.

**Countries/regions locations:** Define a set of countries. Based on the IP-to-country mapping used by Entra ID. A sign-in is associated with the country where its source IP geolocates.

The difference: IP ranges are precise (you know exactly which IPs are trusted). Country locations are approximate (geoIP mapping is not 100% accurate, VPNs can spoof location).

> 🔑 **Key concept:** Marking an IP range as **trusted** does not automatically reduce what CA policies require. Trust means Entra ID treats that location as "known" in risk scoring — it does not bypass CA policies. A policy that requires MFA for All users from All locations still requires MFA from trusted networks. You must explicitly write a condition to grant less friction from trusted locations.

---

## 🌍 How CA Evaluates Location

When a sign-in occurs, CA checks:

1. Does the source IP match any IP-range Named Location?
2. If yes: is that Named Location marked as trusted?
3. What country does the IP geolocate to?
4. Does any policy have a location condition?
5. If yes: does this sign-in's location match the Include or Exclude for that policy?

A policy with **Locations → Include: All trusted locations** applies to sign-ins from IP ranges marked as trusted. A policy with **Locations → Include: All locations, Exclude: All trusted locations** applies to sign-ins from everywhere except trusted networks.

---

## Lab 4.4.1 — Create a Trusted IP-Range Named Location

**Tools:** Entra admin center
**Prerequisite:** Security Defaults disabled (Module 4.1)

1. First, find your current public IP address. Open a browser tab and navigate to `whatismyip.com` or `ifconfig.me`. Note the IP address shown.

2. Open the Entra admin center at `entra.microsoft.com`. Sign in as your Global Administrator.

3. Navigate to **Protection** → **Conditional Access** → **Named locations**.

4. Click **+ New location** → **IP ranges location**.

5. Configure:
   - **Name:** `Contoso-Trusted-Network`
   - **Mark as trusted location:** Enable this toggle
   - **IP ranges:** Click **+** to add a range. Enter your current IP address in CIDR notation: `[your-ip]/32` (the `/32` means exactly one address). Click **Add**.

6. Click **Create**.

7. The location `Contoso-Trusted-Network` appears in the list with a **Trusted** indicator. Any sign-in from your current IP address will now be associated with this location.

---

## Lab 4.4.2 — Create a Countries Named Location

**Tools:** Entra admin center
**Prerequisite:** Lab 4.4.1 complete

1. Navigate to **Protection** → **Conditional Access** → **Named locations**.

2. Click **+ New location** → **Countries/regions location**.

3. Configure:
   - **Name:** `Contoso-Allowed-Countries`
   - **Determine location by:** Leave at **IP address** (the default — uses IP-to-country mapping)
   - **Countries/regions:** Search for and select **United States**. Add any other countries relevant to your scenario.

4. Click **Create**.

You now have two named locations: a trusted IP range and a list of allowed countries.

---

## Lab 4.4.3 — Build a Location-Based CA Policy

**Tools:** Entra admin center
**Prerequisite:** Labs 4.4.1 and 4.4.2 complete

You will build a policy that blocks sign-ins from countries outside Contoso-Allowed-Countries. This is the "geo-block" policy that prevents sign-ins from countries where Contoso has no business presence.

1. Navigate to **Protection** → **Conditional Access** → **Policies** → **+ New policy**.

2. Name the policy: `Block sign-ins from disallowed countries`

**Users:**

3. Include: **All users**

4. Exclude: **CA-Exclusion-BreakGlass** group

**Target resources:**

5. Include: **All cloud apps**

**Conditions:**

6. Click **0 conditions selected**. Select **Locations**.

7. Set **Configure** to **Yes**.

8. Under **Include**: select **Any location**.

9. Under **Exclude**: select **Selected locations** → **Contoso-Allowed-Countries**.

10. Click **Done**.

The logic: Include all locations, Exclude allowed countries = Block only from outside the allowed countries.

**Grant:**

11. Select **Block access**.

**State:**

12. Set to **Report-only** — DO NOT enable this as Enforce until you have verified your sign-in location is in the allowed countries list. If your IP geolocates outside the United States and you enforce a block on non-US locations, you will block yourself.

13. Click **Create**.

**Verify using What-If:**

14. Navigate to the **What If** tool in Conditional Access.

15. Set User to **Alex Lee**, Location to **United States** (or your current country). Click **What If**. The policy should show as **Report-only — would block** or not apply based on whether the location matches.

16. Change Location to a country not in `Contoso-Allowed-Countries`. Click **What If** again. The policy should show it would block from that location.

---

## Scenario Check

Contoso now has two Named Locations defined. The office network IP range is marked as trusted — sign-ins from this range carry a "known location" signal in Identity Protection risk scoring. The allowed countries list contains the United States, where Contoso operates.

The country-block policy is running in Report-Only. It would block any sign-in from outside the United States — including from VPNs that exit in other countries or from actual foreign sign-ins. Before enforcing it, Jordan Kim needs to verify that any remote workers or travel scenarios are covered (VPN IP exit locations may need to be added to `Contoso-Trusted-Network` or the allowed countries list updated).

Named Locations are now available as conditions in future policies — the device compliance and app condition policies in Modules 4.5 and 4.6 can reference them.

---

## Check Your Understanding

- [ ] Navigate to **Named locations** — confirm both `Contoso-Trusted-Network` and `Contoso-Allowed-Countries` appear
- [ ] Use the What-If tool: simulate Alex signing in from your current location — confirm the country-block policy shows as Report-only (not blocked)
- [ ] From memory: explain the difference between marking a location as trusted vs using it as an Exclude in a CA condition
- [ ] State the CA logic: Include all locations + Exclude specific countries = what does this policy target?
- [ ] Explain why the country-block policy should not be switched to Enforce until the VPN exit IPs are verified
- [ ] Name the two types of Named Locations and what each one is based on

---

## Common Mistakes

**Enforcing a country-block policy before adding VPN exit IPs.**
Remote workers using a corporate VPN may have their traffic exit in a different country. If that country is not in the allowed list and the policy is enforced, those workers are blocked. Always audit VPN exit locations before enforcing geo-block policies.

**Marking a location as trusted and expecting it to bypass CA policies.**
Trust is a signal — not an exemption. A trusted location does not bypass CA policies. To reduce friction for trusted networks (such as not requiring MFA from the office), you must explicitly configure that in the policy conditions. "Trusted" affects risk scoring, not policy enforcement.

**Using `/32` for a dynamic IP address.**
ISPs and corporate networks often use dynamic IP addresses that change. A `/32` CIDR (single IP) will break the named location when the IP changes. For office networks, use the full IP range your ISP assigned. For home or developer tenant testing, the `/32` is fine for a lab exercise.

**Creating a location Named Location using country without verifying geoIP accuracy.**
IP-to-country geolocation is not always accurate. Some VPNs and satellite internet providers may geolocate to unexpected countries. Test the country detection by checking sign-in logs for the country shown on actual sign-ins before building policies that rely on country accuracy.

---

## What's Next

Module 4.5 covers device conditions in Conditional Access — requiring a managed, compliant device for access to sensitive resources. Named Locations from this module can be combined with device conditions: require a compliant device when signing in from an untrusted location, but allow just MFA from a trusted network.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.3 — Your First CA Policy](../4-3-your-first-ca-policy/4-3-your-first-ca-policy.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 4.5 — Device Conditions →](../4-5-device-conditions/4-5-device-conditions.md)
