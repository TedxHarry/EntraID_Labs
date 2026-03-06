# Access Packages — Build Resource Bundles for Users to Request
*Group related resources (groups, apps, sites) into packages so users can request them together.*

> 🟡 **Intermediate** · Unit 8, Module 8.2 · ⏱️ 50 minutes

---

## What You Will Learn

- Create access packages in entitlement management
- Add resources (groups, apps, SharePoint sites) to packages
- Configure access package settings (who can request, time limits, renewals)
- Set resource roles (member vs. owner)
- Publish packages to a catalog

---

## Why This Matters

*SC-300: Plan and automate identity governance — Create and manage access packages*

An access package is where entitlement management becomes real. Instead of thinking about "access" abstractly, you define concrete packages of resources. "Finance Team Access" includes specific groups, apps, and sites. This is what users request, managers approve, and admins maintain. Well-designed packages make governance scalable.

---

## 🏗️ Building Access Packages

### Step 1: Create a Catalog
A **catalog** is a container for related access packages. Think of it as a folder.

**Examples:**
- "Contoso Public Catalog" — Everyone can browse
- "Finance Catalog" — Finance-specific packages
- "HR Catalog" — HR and recruitment packages

### Step 2: Create Access Package
Within a catalog, create packages.

**Examples:**
- "Finance Team Access" — Finance team members
- "Finance Manager Access" — Finance managers only
- "Temporary Project Access" — Project-specific, short-term

### Step 3: Add Resources to Package
Define what users get when they're granted the package.

**Resources can be:**
- **Security groups** (get membership)
- **Microsoft 365 groups** (get membership, collaborative groups)
- **Apps** (get assignment)
- **SharePoint sites** (get site membership)
- **Custom roles** (delegated access)

### Step 4: Configure Settings
Set time limits, renewal policies, approval requirements.

### Step 5: Publish Package
Make it available to users for requests.

---

## Lab 8.2.1 — Create a Catalog

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Module 8.1 complete

Goal: Create a catalog for Finance team access packages.

1. Navigate to **Identity Governance** → **Entitlement Management** → **Catalogs**.

2. Click **+ New catalog** (or **Create new catalog**).

3. Fill in the details:
   - **Name:** `Finance Catalog`
   - **Description:** `Access packages for Finance team members`
   - **Enabled:** Yes (catalog is active for requests)

4. Click **Create**.

**Result:** Catalog is created. Now you can add packages to it.

---

## Lab 8.2.2 — Create Your First Access Package

**Time:** 20 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 8.2.1 complete

Goal: Create "Finance Team Access" package.

### Step 1: New Access Package

1. Navigate to **Entitlement Management** → **Access packages**.

2. Click **+ New access package**.

3. Fill in basics:
   - **Name:** `Finance Team Access`
   - **Description:** `Access for Finance team members: shared folders, Finance apps, Finance group`
   - **Catalog:** Select `Finance Catalog` (created in 8.2.1)

4. Click **Next** (or continue).

---

### Step 2: Add Resources

1. You'll see **Add resources to package** (or **Add resource roles**).

2. Click **+ Add resource roles** (or **+ Add resources**).

3. **Select resources to add:**

   **Resource 1: Finance-All Group**
   - Type: Security group
   - Search for **Finance-All** (or create it if it doesn't exist)
   - Select: **Finance-All**
   - Role: **Member** (users get membership in the group)

   **Resource 2: Finance SharePoint Site**
   - Type: SharePoint site
   - Search for **Finance** shared site
   - Select: Finance site
   - Role: **Member**

   **Resource 3: Finance Apps (Contoso Financial System)**
   - Type: Application
   - Search for **Finance**
   - Select the Finance app
   - Role: **User** (or whatever role the app offers)

4. After adding resources, you'll see them listed:
   - Finance-All (Group) — Member role
   - Finance SharePoint (Site) — Member role
   - Finance App (Application) — User role

5. Click **Next**.

---

### Step 3: Configure Request Settings

1. **Who can request?**
   - Option A: "All members" — Any user in your tenant
   - Option B: "For specific users" — Only certain groups (e.g., "All Finance employees")
   - Option C: "Admin assignment only" — Only admins can assign (no self-service)

   For Finance, select: **All members** (anyone can request, but it has to be approved based on role)

2. **Require justification?**
   - Yes (users must provide reason: "Why do you need Finance access?")

3. **Multi-stage approval?**
   - Skip for now (you'll configure approval rules next module)

4. Click **Next**.

---

### Step 4: Configure Lifetime Settings

1. **Duration after assignment:**
   - Select: **Custom**
   - **Duration:** 90 days (standard review cycle)

2. **Allow renewal?**
   - Yes (users can renew before expiration)

3. **Days before expiration to request renewal:**
   - 30 days (user gets notified to renew 30 days before expiration)

4. **Access review?**
   - Optional: Enable "Require access review" (forces manager to review every 90 days)

5. Click **Next**.

---

### Step 5: Publish Package

1. You'll see a summary:
   - Package name: Finance Team Access
   - Catalog: Finance Catalog
   - Resources: Finance-All (Group), Finance SharePoint (Site), Finance App
   - Lifetime: 90 days with renewal allowed
   - Requesters: All members

2. Click **Create** (or **Publish**).

**Result:** Package is created and published. Users can now request it.

---

## Lab 8.2.3 — Verify Package Configuration

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 8.2.2 complete

Verify that the package is configured correctly.

1. Navigate to **Entitlement Management** → **Access packages**.

2. Find **Finance Team Access**.

3. Click on it to view details.

4. Review sections:
   - **Overview** — Name, description, catalog
   - **Resource roles** — Resources included
   - **Request settings** — Who can request, justification required?
   - **Lifecycle** — Duration, renewal rules
   - **Assignments** — Users who have been granted this package (should be empty initially)

5. Verify everything looks correct:
   - Resources are listed ✓
   - Duration is 90 days ✓
   - Renewal is enabled ✓
   - Requesters are "All members" ✓

---

## Lab 8.2.4 — Create a Second Package (Manager Access)

**Time:** 15 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 8.2.3 complete

Create a more restrictive package for Finance managers.

### Design:

**Package:** Finance Manager Access
**Resources:**
- Finance-Managers group (higher privileges)
- Finance Reports app (reporting only for managers)
- Finance Analytics dashboard (manager-specific analytics)

**Settings:**
- **Who can request?** Only Finance managers (not everyone)
- **Justification?** Yes
- **Duration:** 180 days (longer, since it's more stable)
- **Renewal?** Yes

### Steps:

1. Create new access package:
   - Name: `Finance Manager Access`
   - Catalog: `Finance Catalog`
   - Add resources: Finance-Managers group, Finance Reports app, Finance Analytics

2. Configure request settings:
   - Who can request: Select **"For specific users"** → Choose Finance managers group
   - Require justification: Yes

3. Configure lifetime:
   - Duration: 180 days (6 months, less frequent reviews)
   - Renewal: Yes

4. Publish.

**Result:** Two packages in Finance Catalog:
- Finance Team Access (90 days, anyone can request, but needs approval)
- Finance Manager Access (180 days, managers only, needs approval)

---

## Lab 8.2.5 — Test Self-Service Request (As User)

**Time:** 10 minutes
**Tools:** My Access portal
**Prerequisite:** Lab 8.2.4 complete

Scenario: You are Alex (Finance employee). Test requesting access.

1. Navigate to **My Access** portal: `https://myaccess.microsoft.com` (or search "My Access").

2. You'll see:
   - **Approved access** — Your current access packages
   - **Request access** — Available packages to request

3. Under **Request access**, you should see:
   - **Finance Team Access** (if requesters = "All members")
   - Maybe **Finance Manager Access** (if you're in the managers group)

4. Click **Finance Team Access** → **Request**.

5. Fill in:
   - **Duration:** How long do you need? (default is package duration: 90 days)
   - **Business justification:** "I'm joining the Finance team"

6. Click **Submit** (or **Request**).

**Result:** Request is submitted. You'll see "Your request has been submitted for approval."

Next module shows how managers approve this request.

---

## Lab 8.2.6 — Manage Package Lifecycle

**Time:** 10 minutes
**Tools:** Entra admin center
**Prerequisite:** Lab 8.2.5 complete

Understand how to modify and decommission packages.

### Scenario: Finance Team Access Needs Update

New Finance app is deployed. "Finance Team Access" package should include it.

1. Navigate to **Entitlement Management** → **Access packages** → **Finance Team Access**.

2. Find **Resource roles** section.

3. Click **+ Add resource roles** (or **Edit resources**).

4. Add the new Finance app.

5. Click **Save**.

**Result:** New users requesting "Finance Team Access" will get the new app included. Existing users won't be updated automatically (they keep their current assignment until renewal).

---

### Decommission Package (When No Longer Needed)

1. On the package page, look for **Delete** or **Archive** option.

2. Click **Delete** (or **Archive**).

3. Users can no longer request the package. Existing users keep their access until expiration (then it's revoked).

---

## The Scenario Check

**Contoso's Finance packages:**

Jordan Kim (Finance Manager) designed two access packages:

1. **Finance Team Access** (90 days, renewable)
   - Finance-All group, Finance SharePoint, Finance app
   - Anyone can request, needs manager approval
   - Used for: Finance team members, finance analysts, temporary finance staff

2. **Finance Manager Access** (180 days, renewable)
   - Finance-Managers group, Finance Reports, Finance Analytics
   - Only managers can request, needs director approval
   - Used for: Finance managers and team leads

When Alex was hired, she requested "Finance Team Access." River approved. Alex received group membership, SharePoint access, and Finance app assignment. After 90 days, Alex renewed the access.

When a contractor (Sam) worked on a 3-month project, Sam requested "Finance Team Access" with justification "3-month contract on Q2 reconciliation." River approved for 90 days. After 90 days, Sam's access automatically expired.

---

## Check Your Understanding

- [ ] Describe the structure of an access package (catalog → package → resources)
- [ ] List three types of resources that can be added to an access package
- [ ] Explain the difference between "Member" and "Owner" roles in a group resource
- [ ] State why time-limited access (90 days) is better than permanent access
- [ ] Design an access package for a 20-person engineering team

---

## Common Mistakes

**Creating packages that are too granular.**
You create "SharePoint Group Access," "Microsoft 365 Group Access," "Security Group Access" (3 packages for groups). Users are confused which one to request. Combine related resources: "Team Member Access" includes all group types needed.

**Forgetting to add resources.**
You create "Finance Team Access" but forget to add the Finance SharePoint site. Users request it, get approved, but don't get the site. Review the package: did you add everything the user needs?

**Setting duration too long.**
"Finance Team Access" has no expiration (permanent). User leaves Finance, no one removes access. Set expiration: 90 days is standard. Forces renewal as a review gate.

**Not enabling renewal.**
Access expires after 90 days. User loses access. Can't renew. User is frustrated. Enable renewal: let active users keep access if it's still needed.

**Updating package retroactively.**
You add a new resource to "Finance Team Access." Existing users don't get it. Only new users do. Consider: Should existing users get the new resource? If yes, manually grant it. If no, document why.

---

## What's Next

Module 8.3 covers **Approval Workflows** — how to configure who approves access requests and set conditions (manager approval for standard requests, director approval for manager packages, etc.).

---

> 📚 **Entra ID — Zero to Admin** · Unit 8: Identity Governance
>
> [← Module 8.1 — Entitlement Management Overview](../8-1-entitlement-management-overview/8-1-entitlement-management-overview.md) | [🏠 Series Home](../../TABLE-OF-CONTENTS.md) | [Module 8.3 — Approval Workflows →](../8-3-approval-workflows/8-3-approval-workflows.md)
