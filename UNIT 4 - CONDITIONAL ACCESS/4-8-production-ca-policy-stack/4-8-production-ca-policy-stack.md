# Production CA Policy Stack — Putting It All Together
*This is not a feature. This is your defense system. Build it properly.*

> 🟡 **Intermediate** · Unit 4, Module 4.8 · ⏱️ 90 minutes

---

## What You Will Learn

- How to design a **realistic, multi-policy CA strategy** based on app sensitivity and risk
- How to implement the **Contoso production policy stack** — MFA, device compliance, geo-blocking, and app-based controls
- How to **measure and manage policy impact** over time using sign-in logs and metrics
- How to **handle exceptions safely** — break-glass access, contractor/partner accounts, legacy systems
- How to **monitor and audit** CA policies — ensuring they are working as intended and policies are not accidentally disabled

---

## Why This Matters

*SC-300: Implement authentication and access management — Design and implement a complete, production-grade CA strategy*

Contoso has 50 people. If you get CA right, they work securely with zero friction. If you get it wrong, you either:
- **Over-lock:** Policies so strict that 10% of users cannot do their jobs. Support gets slammed with "I can't access SharePoint" tickets. You get pressure to disable policies.
- **Under-lock:** Policies so weak that they are useless. A breach happens, and you realize your policies would not have stopped it.

The goal is **the middle ground:** policies that **actually reduce risk** (stop real attacks) without breaking legitimate work. This module teaches you how to design that balance.

---

## 🏗️ The Contoso CA Stack Design

By the end of Module 4.7, Contoso has:
1. ✅ Security Defaults disabled (Module 4.1) — CA policies now control security
2. ✅ MFA policy for administrators (Module 4.3) — Enforce
3. ✅ Geo-block policy (Module 4.4) — Report-Only, showing minimal impact
4. ✅ Device compliance policy for SharePoint (Module 4.5) — Report-Only, showing 35% impact
5. ✅ App-based policies for Azure Portal (phishing-resistant) and other apps (standard MFA) (Module 4.6) — Both Report-Only
6. ✅ Understanding of Report-Only data and testing (Module 4.7) — Ready to enforce safely

**The Production Stack** (this module) will:
- Move the geo-block policy to **Enforce** (low risk, high value)
- Plan the device compliance rollout (needs user enrollment time)
- Plan the phishing-resistant MFA rollout for Azure Portal (needs Windows Hello enrollment)
- Add **exclusion groups** for contractors, partners, and special cases
- Add **monitoring and alerting** to catch policy misconfigurations
- Document the strategy and create a 12-week rollout plan

---

## 📊 The Three Dimensions of CA Policy Design

**Dimension 1: User Sensitivity (Who?)**
- **Standard users** — general employees (Teams, email, OneDrive)
- **Administrators** — admins, IT staff (access to Azure portal, Intune, compliance tools)
- **Contractors/Partners** — external users (limited access, lighter controls)
- **Service principals** — applications, automation (treated separately, not in this module)

**Dimension 2: App Sensitivity (What?)**
- **Tier 1 (Low):** Teams, Skype, Yammer, OneDrive personal
- **Tier 2 (Medium):** SharePoint, Exchange, Project
- **Tier 3 (High):** Azure Portal, Entra Admin Center, Azure Management APIs
- **Tier 4 (Critical):** PIM (Privileged Identity Management), Identity Protection

**Dimension 3: Condition Sensitivity (How much control?)**
- **Location-based:** Block high-risk countries, require MFA from untrusted networks
- **Device-based:** Require compliant devices for sensitive apps
- **Authentication-based:** Require phishing-resistant MFA for admin portals
- **Risk-based:** Block impossible travel, suspicious sign-in patterns (covered in Unit 8)

The CA stack combines these dimensions: "Administrators + Tier 3 apps + from untrusted location" gets the strictest controls.

---

## 🎯 Contoso's Production Policy Stack

Here is what Contoso will implement and enforce:

### Policy 1: Block Sign-Ins from Disallowed Countries (Enforce)
**Current state:** Report-Only (from Module 4.4)
**Target state:** Enforce
**Justification:** Low impact (0-5% users), high value (blocks foreign threats)
**User group:** All users
**Apps:** All cloud apps
**Condition:** Location = Outside allowed countries
**Grant:** Block access

### Policy 2: Require MFA for Administrators (Enforce)
**Current state:** Enforce (from Module 4.3, enabled in Module 4.7)
**User group:** All users with Administrator role
**Apps:** All cloud apps
**Condition:** None (applies to all access)
**Grant:** Require MFA

### Policy 3: Require Compliant Device for SharePoint (Enforce in Week 3)
**Current state:** Report-Only (from Module 4.5)
**Target state:** Enforce (after user enrollment)
**Justification:** Finance team works with sensitive budgets; device compliance required
**User group:** All users (except Contractors-NoCompliance exclusion)
**Apps:** Office 365 SharePoint Online
**Condition:** None
**Grant:** Require device to be compliant
**Exclusion:** Contractors-NoCompliance (contractors cannot enroll in Intune, so they are excluded)

### Policy 4: Require Phishing-Resistant MFA for Azure Portal (Report-Only for Week 1-3, Enforce in Week 4)
**Current state:** Report-Only (from Module 4.6)
**Target state:** Enforce (after Windows Hello enrollment for admins)
**Justification:** Azure Portal is high-risk; compromised admin = compromised tenant
**User group:** Users with Administrative roles
**Apps:** Azure Portal, Azure Management APIs
**Condition:** None
**Grant:** Require Authentication Strength = Phishing-resistant MFA
**Exclusion:** Emergency-Access (break-glass account, does not have phishing-resistant MFA; exclusion ensures break-glass is always accessible)

### Policy 5: Require MFA for All Other Apps (Enforce in Week 2)
**Current state:** Report-Only (from Module 4.6)
**Target state:** Enforce
**Justification:** Standard baseline for all applications
**User group:** All users
**Apps:** Teams, SharePoint, Exchange, OneDrive, Power BI
**Condition:** None
**Grant:** Require MFA

---

## 📈 The 12-Week Rollout Plan

| Week | Action | Impact |
|---|---|---|
| **Week 1** | Enforce geo-block policy. Collect data on device compliance and phishing-resistant MFA policies (Report-Only). | Blocks ~5 foreign attacks, zero legitimate users blocked |
| **Week 2** | Start user communication: "Device compliance is coming to SharePoint in 4 weeks. Enroll your device in Intune now." Begin Intune enrollment for pilot group (IT department). Enforce "Require MFA for all other apps" (Teams, Exchange, OneDrive). | Communication sent, pilot enrolls, standard MFA enforced |
| **Week 3** | Week 2 pilot reports: 95% of IT staff enrolled in Intune and compliant. Expand enrollment to Finance department. Enforce "Require compliant device for SharePoint." | Finance team enrolls, SharePoint now requires device compliance |
| **Week 4** | IT department reports: 90% of full staff enrolled. Start Windows Hello enrollment for administrators. Enforce phishing-resistant MFA for Azure Portal (Report-Only data shows zero impact if admins have Windows Hello). | Admin enrollment campaign begins |
| **Week 5-8** | Windows Hello enrollment continues. Contractors are added to Contractors-NoCompliance exclusion (for device compliance policy) and to Contractors-NoPhishingResistant exclusion (for Azure Portal policy). Monitor sign-in logs for policy violations. | Contractors excluded, monitoring active |
| **Week 9-12** | Policies fully enforced. Measure impact: review sign-in logs, support tickets, and policy evaluation results. Adjust exclusions as needed. Plan Unit 5 (Identity Protection and risk-based policies). | Policies stable, data collected |

---

## Lab 4.8.1 — Design Your Exclusion Groups (Strategic Planning)

**Time:** 15 minutes
**Tools:** Entra admin center, spreadsheet or text editor
**Prerequisite:** All Module 4 labs complete

Before enforcing policies, you must identify who needs exclusions. Create exclusion groups for:

1. **CA-Exclusion-BreakGlass** (created in Module 4.3 — already exists)
   - Member: Your Emergency Access Account (Global Administrator, no MFA configured)
   - Excluded from: All CA policies
   - Reason: This account must always be accessible, even if policies fail

2. **Contractors-NoCompliance** (new group for this lab)
   - Members: Contractor accounts, partner accounts, anyone who cannot enroll in Intune
   - Excluded from: Device compliance policies
   - Reason: Contractors often use personal devices; forcing Intune enrollment is not feasible
   - **Policies affected:** "Require compliant device for SharePoint"

3. **Contractors-NoPhishingResistant** (new group for this lab)
   - Members: Same as Contractors-NoCompliance
   - Excluded from: Phishing-resistant MFA policies
   - Reason: Contractors do not have Windows Hello or FIDO2 keys configured
   - **Policies affected:** "Require phishing-resistant MFA for Azure Portal"

4. **ServiceAccounts-Excluded** (optional for future)
   - Members: Service principals, automation accounts, legacy systems
   - Excluded from: MFA policies
   - Reason: Service accounts cannot provide MFA; they authenticate with client credentials
   - **Note:** This is advanced and covered in Unit 5. For now, just plan it.

**Task:** Create these groups in your tenant. You will reference them when enforcing policies.

### Create CA-Exclusion-BreakGlass (if not already exists):

1. Open the Entra admin center. Navigate to **Groups** → **All groups** → **+ New group**.

2. Configure:
   - **Group type:** Security
   - **Group name:** `CA-Exclusion-BreakGlass`
   - **Members:** Click **No members selected** → Search for your Emergency Access Account → Add

3. Click **Create**.

### Create Contractors-NoCompliance:

4. Repeat the process:
   - **Group name:** `Contractors-NoCompliance`
   - **Members:** Add any contractor or partner accounts you have. (If you do not have real contractors, create a test user named "Test Contractor" and add them.)

5. Click **Create**.

### Create Contractors-NoPhishingResistant:

6. Repeat:
   - **Group name:** `Contractors-NoPhishingResistant`
   - **Members:** Same as Contractors-NoCompliance

7. Click **Create**.

**Verification:**

8. Navigate to **Groups** and confirm all three groups exist:
   - ✅ CA-Exclusion-BreakGlass
   - ✅ Contractors-NoCompliance
   - ✅ Contractors-NoPhishingResistant

---

## Lab 4.8.2 — Update Existing Policies with Exclusions

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 4.8.1 complete

Now you will update the CA policies to exclude the appropriate groups.

### Update: "Require compliant device for SharePoint" (from Module 4.5)

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. Find and open **"Require compliant device for SharePoint"**.

3. Under **Users**, go to **Exclude**.

4. Click **0 items selected** (in Exclude section).

5. Select **Groups** → Search for `Contractors-NoCompliance` → Add.

6. Click **Select**.

7. Click **Save**.

**Result:** Contractors-NoCompliance group is now excluded. When a contractor signs into SharePoint, the device compliance policy will not apply.

### Update: "Require phishing-resistant MFA for Azure Portal" (from Module 4.6)

8. Find and open **"Require phishing-resistant MFA for Azure Portal"**.

9. Under **Users**, go to **Exclude**.

10. Click **0 items selected**.

11. Select **Groups** → Search for `Contractors-NoPhishingResistant` → Add.

12. Also add **CA-Exclusion-BreakGlass** (to exclude the break-glass account).

13. Click **Select** → **Save**.

**Result:** Both groups are excluded. Contractors and the break-glass account will not be subject to this policy.

### Update: "Block sign-ins from disallowed countries" (from Module 4.4)

14. Find and open **"Block sign-ins from disallowed countries"**.

15. Under **Users**, verify that **CA-Exclusion-BreakGlass** is excluded (it should be from Module 4.4).

16. If not, add it now.

17. Click **Save**.

**Result:** All critical policies now have appropriate exclusions.

---

## Lab 4.8.3 — Enforce the Geo-Block Policy (Lowest Risk, High Value)

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 4.8.2 complete

The geo-block policy from Module 4.4 is Report-Only with minimal impact (blocking only foreign attacks). This is safe to enforce.

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. Find and open **"Block sign-ins from disallowed countries"**.

3. Under **State**, change from **Report-only** to **On** (Enforce).

4. Click **Save**.

**Verification:**

5. Navigate to **Sign-in logs**.

6. Look for a recent sign-in from a user outside your allowed countries (if available). The policy status should now show:
   - **Status:** Enforced
   - **Result:** Denied (actually blocked, not Report-only anymore)

7. If you see foreign sign-ins being blocked, the policy is working.

---

## Lab 4.8.4 — Plan Device Compliance Rollout (Metrics & Timeline)

**Time:** 15 minutes
**Tools:** Entra admin center, sign-in logs analysis
**Prerequisite:** Lab 4.8.3 complete

The device compliance policy for SharePoint is Report-Only. Enforcing it requires careful planning because many users are not compliant yet.

1. Navigate to **Protection** → **Conditional Access** → **Sign-in logs**.

2. Filter for SharePoint sign-ins in the last 7 days.

3. For each sign-in, look at the **Device Compliance Status**:
   - Count how many are **Compliant**
   - Count how many are **Non-compliant**
   - Count how many are **Not registered**

4. Calculate the percentage:
   - **% Compliant** = (Compliant sign-ins / Total sign-ins) × 100
   - Example: 35 Compliant out of 100 SharePoint sign-ins = 35% compliant

5. **Decision logic:**
   - **>80% compliant:** Safe to enforce. You would block only 20% of users; most are already compliant.
   - **60-80% compliant:** Medium risk. Plan 1-2 weeks of user enrollment before enforcing.
   - **<60% compliant:** High risk. Users need 3-4 weeks to enroll. If you enforce now, support will be overwhelmed.

6. **For Contoso scenario:** Assume you find 35% compliant (based on Module 4.5 lab). Decision: Do NOT enforce yet. Plan a 2-week enrollment campaign, then re-measure. Target: get to 75%+ compliant before enforcing.

7. Document your plan:
   ```
   Device Compliance Rollout Plan for SharePoint:
   - Current compliant: 35%
   - Target before enforcement: 75%
   - Timeline: 2 weeks for user enrollment
   - Week 1: User communication, enrollment campaign begins
   - Week 2: IT team follows up with non-compliant users, remediation
   - Week 3: Enforce policy (expected impact: ~20-25% of users)
   - Week 4: Monitor, handle exceptions, document results
   ```

---

## Lab 4.8.5 — Audit Your CA Policies (Comprehensive Review)

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** All previous labs complete

Before you consider your CA stack production-ready, perform a comprehensive audit.

### Checklist 1: All Policies Configured Correctly

1. Navigate to **Protection** → **Conditional Access** → **Policies**.

2. For each policy, verify:
   - [ ] **Name is clear** — can you tell what it does by reading the name?
   - [ ] **Users configured:** Who does it apply to? (All, specific group, admin role)
   - [ ] **Exclusions are set:** CA-Exclusion-BreakGlass is excluded (if applicable)
   - [ ] **Apps configured:** Which apps does it target? (All, specific apps)
   - [ ] **Conditions are correct:** Location, device, MFA method (as intended)
   - [ ] **Grant is set:** Allow, Block, or Require control
   - [ ] **State is correct:** Report-only or Enforce (as planned)

3. Create a simple table:

   | Policy Name | Users | Excluded | Apps | Conditions | Grant | State | Status |
   |---|---|---|---|---|---|---|---|
   | Block disallowed countries | All | BreakGlass | All | Location | Block | Enforce | ✅ |
   | MFA for admins | Admin role | BreakGlass | All | None | MFA | Enforce | ✅ |
   | Compliant device for SharePoint | All | Contractors | SharePoint | None | Compliant | Report-only | ⏳ |

4. If any policy has a ❌, fix it before enforcement.

### Checklist 2: Exclusion Groups Exist and Are Used Correctly

5. Navigate to **Groups** → **All groups**.

6. Verify these groups exist and are not empty:
   - [ ] CA-Exclusion-BreakGlass (should contain your emergency access account)
   - [ ] Contractors-NoCompliance (should contain contractor accounts)
   - [ ] Contractors-NoPhishingResistant (should contain contractor accounts)

7. Verify each exclusion group is used in at least one CA policy (check each policy's Exclude section).

### Checklist 3: Report-Only Policies Have Data

8. Navigate to **Sign-in logs**.

9. For each Report-Only policy, check:
   - [ ] Policy appears in recent sign-in logs
   - [ ] Results show a mix of "Satisfied," "Would Block," "Would require auth"
   - [ ] No unexpected errors or evaluation failures

10. If a Report-Only policy has not evaluated any sign-ins in 7 days, it might be targeting an app or user that is not being used. Review the policy logic.

### Checklist 4: No Policies Are Accidentally Disabled

11. Back in **Conditional Access** → **Policies**, look for policies with a **⚠️ warning** icon or **disabled** status.

12. Verify every policy is either **Report-only** or **On** (never accidentally disabled).

13. If a policy is disabled, re-enable it (unless intentionally disabled for testing).

### Checklist 5: Break-Glass Exclusion Is Correct

14. For every Enforce policy, verify **CA-Exclusion-BreakGlass** is excluded:
   - [ ] "Block disallowed countries" — BreakGlass excluded? Yes / No
   - [ ] "MFA for admins" — BreakGlass excluded? Yes / No

15. If a critical policy does not exclude the break-glass account, add it now.

---

## Lab 4.8.6 — Monitor and Document (Ongoing)

**Time:** 15 minutes
**Tools:** Sign-in logs, spreadsheet
**Prerequisite:** Lab 4.8.5 complete

Create a monitoring routine that you will do weekly.

**Weekly CA Policy Monitoring Checklist:**

1. **Check for policy denials:**
   - Navigate to **Sign-in logs**
   - Filter for denials in the last 7 days
   - Look for CA policies that are blocking legitimate users
   - If you find a block that should not happen, create an exclusion

2. **Check for unexpected policies in logs:**
   - Are there Report-Only policies showing results you did not expect?
   - Are there policies NOT in the logs (targeting inactive users/apps)?

3. **Check support tickets:**
   - Are users reporting "I cannot access [app]" problems?
   - Correlate with CA policy blocks

4. **Check for policy conflicts:**
   - Are two policies conflicting? (E.g., one requires MFA, another blocks from untrusted location — both must be satisfied)
   - Review What-If tool to resolve conflicts

5. **Document changes:**
   - If you add a new exclusion, update your exclusion group list
   - If you change a policy, log the change and reason

**Create a simple monitoring log (template):**

```
CA Policy Monitoring Log

Week of March 1:
- MFA for admins: 0 blocks, all satisfied ✅
- Geo-block: 3 blocks from China, 2 blocks from Russia ✅ (expected)
- Device compliance SharePoint: Report-Only, 40% would block ⏳ (monitoring for enforcement)
- Phishing-resistant MFA Azure Portal: Report-Only, 85% would block ⏳ (Windows Hello enrollment in progress)
- Notes: No support tickets related to CA policies. Contractor accounts are correctly excluded.
```

---

## Scenario Check

Contoso's production CA stack is now defined, partially enforced, and being monitored.

**Current state (End of Week 1):**
- ✅ Geo-block policy **Enforce** — blocking foreign attacks
- ✅ MFA for admins **Enforce** — zero impact (admins already using MFA)
- ✅ All other policies **Report-Only** — collecting data

**Timeline ahead (Weeks 2-12):**
- Week 2: Enforce standard MFA for Teams, Exchange, OneDrive
- Week 3: Device compliance for SharePoint enforcement (after user enrollment)
- Week 4: Phishing-resistant MFA for Azure Portal (after Windows Hello deployment)
- Weeks 5+: Monitoring, adjusting, and planning next unit (Identity Protection)

Jordan Kim (security architect) now has a documented, measurable CA strategy. Not just policies, but a **business-aware rollout plan** that balances security with user experience.

---

## Check Your Understanding

- [ ] Design a CA policy for a specific scenario: "Protect the Finance department's sensitive data in SharePoint." What conditions would you set? What controls would you require?
- [ ] You have a Report-Only policy showing 60% of users would be blocked. Is it safe to enforce? Why or why not?
- [ ] Explain why the break-glass account must be excluded from all CA policies.
- [ ] You enforce a device compliance policy and 3 contractors cannot access SharePoint. You did not exclude them. What was your mistake?
- [ ] Create a 4-week rollout plan for a new phishing-resistant MFA policy for the Azure Portal (assume 20% of admins have Windows Hello today).
- [ ] In the sign-in logs, you see a policy "Would Block" for a user who should have access. What are two possible causes? How would you fix each?

---

## Common Mistakes

**Enforcing multiple policies in the same week.**
If you enforce Policy A on Monday and Policy B on Tuesday, and something breaks Wednesday, you have two suspects. This makes troubleshooting hard. Always enforce one policy at a time, wait 3-5 days, verify, then enforce the next.

**Not measuring impact before enforcement.**
Policies that look good in design often have unexpected impacts in production. A policy targeting "administrators" might accidentally include more people than you think (if the admin role is assigned broadly). Always run Report-Only first and measure real impact.

**Forgetting to exclude contractors and partners.**
Contractors often cannot enroll in Intune or have phishing-resistant MFA configured. If you enforce a policy without excluding them, they are locked out. Identify external users early and exclude them proactively.

**Creating too many policies.**
Policies stack and conflict. If you have 10 overlapping policies, users are confused about why they are blocked, and troubleshooting is hard. Start simple (one policy per app/condition), then add only as needed.

**Not documenting policy intent.**
Six months from now, someone will ask "Why do we require compliant devices for SharePoint?" If you did not document it (e.g., "Finance data is sensitive, PII requires protection, device compliance is stronger than MFA for local threat mitigation"), the answer is lost. Always document the "why" behind each policy.

**Disabling a policy to "fix" a problem instead of understanding root cause.**
If a policy is blocking legitimate users, do NOT disable it. Instead: (a) check if they should be excluded, (b) check if the policy logic is correct, (c) add a remediation group. Disabling a policy removes the control you were trying to achieve.

---

## What's Next

Unit 4 (Conditional Access) is complete. You have built a production-grade CA strategy for Contoso: geo-blocking, MFA, device compliance, and app-based controls.

**Unit 5: Identity Protection and Risk-Based Access** covers dynamic, risk-aware CA policies. Instead of static rules ("always require MFA"), Unit 5 teaches you to write policies that adapt: "require MFA if sign-in is risky, allow if it is known good." You will learn about sign-in risk, user risk, and how Entra ID's machine learning detects compromised accounts in real time.

By the end of Unit 5, Contoso will have both **preventive CA policies** (Unit 4) and **detective/adaptive CA policies** (Unit 5) — a complete defense system.

---

> 📚 **Entra ID — Zero to Admin** · Unit 4: Conditional Access
>
> [← Module 4.7 — Report-Only Mode and Testing](../4-7-report-only-mode-and-testing/4-7-report-only-mode-and-testing.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Unit 5 — Identity Protection and Risk-Based Access →](../../UNIT%205%20-%20IDENTITY%20PROTECTION/README.md)
