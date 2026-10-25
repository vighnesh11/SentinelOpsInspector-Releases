# Azure and Microsoft Entra ID Integration Guide

Configure **SentinelOps Inspector** to query **your own** Azure Log Analytics workspace with delegated, read-only access.

> **Authentication model:** SentinelOps Inspector is a Windows desktop public-client application. It opens an interactive Microsoft sign-in and uses the permissions of the **signed-in Entra user account**. It does not authenticate as a service principal, does not need a client secret, and does not require Contributor or Owner rights on the workspace.

## Before you begin

Ensure you have:

- A Microsoft Entra tenant where you are authorized to register applications (or an administrator who can help).
- An Azure subscription with a **Log Analytics workspace**.
- A designated **Entra user identity** authorized for interactive sign-in and subject to your organization's MFA and Conditional Access policies. This guide refers to it as the **technical account**; it is *a user*, not a service principal or client-credentials identity.
- Permission to assign users to enterprise applications and permission to create an Azure RBAC role assignment at the target workspace, or assistance from the relevant administrators.
- SentinelOps Inspector installed from the [public Releases page](https://github.com/vighnesh11/SentinelOpsInspector-Releases/releases).

Use an authorized non-production workspace for initial evaluation. Your organization's security policies take precedence.

## 1. Register a Microsoft Entra application

1. Sign in to the [Microsoft Entra admin center](https://entra.microsoft.com/) and go to **Entra ID → App registrations → New registration**.
2. Give the application a descriptive name, such as **SentinelOps Inspector**.
3. Select **Accounts in this organizational directory only** for a tenant-specific setup.
4. Select **Register**.
5. Open the new app registration and record the **Application (client) ID** and **Directory (tenant) ID** shown on its **Overview** page.

### Configure desktop authentication

1. In the app registration, open **Authentication**.
2. Select **Add a platform → Mobile and desktop applications**.
3. Register this redirect URI: `http://localhost`.
4. Under **Advanced settings**, set **Allow public client flows** to **Yes**, then save.

The application uses the system browser and a localhost callback. Do **not** configure it as a confidential web application and do **not** create or share a client secret.

## 2. Add delegated Log Analytics API permission

1. In **Entra ID → App registrations**, open **SentinelOps Inspector**.
2. Select **API permissions → Add a permission**.
3. Open **APIs my organization uses** and search for **Log Analytics API**.
4. Select **Delegated permissions**.
5. Select **Data.Read** and choose **Add permissions**.
6. If your tenant requires administrator consent, have an authorized administrator review the permission and select **Grant admin consent**.

**Required permission:**

| API | Permission type | Permission |
| --- | --- | --- |
| Log Analytics API | Delegated | `Data.Read` |

> **Important:** Delegated `Data.Read` is not sufficient by itself. The signed-in technical account must also have access to the Log Analytics workspace through Azure RBAC (Step 4).

## 3. Assign the technical account to the enterprise application

**This assignment is performed under Enterprise applications, not inside the App registrations page.** An app registration defines the application; its **Enterprise Application (service principal)** is where user access is managed.

1. Go to **Entra ID → Enterprise applications → All applications**.
2. Find the **SentinelOps Inspector** enterprise application created for your registration.
3. For restricted access, open **Properties** and set **Assignment required? = Yes**, then save. This setting is optional unless you specifically want to restrict access to assigned users, but it is recommended for a controlled technical-account setup.
4. Select **Users and groups → Add user/group**.
5. Find and select the Entra **user account** that will sign in to SentinelOps Inspector.
6. For **Select a role**, choose **Default Access** if no custom app roles exist.
7. Select **Assign**.

If assignment is required, your administrator must also grant the necessary consent to the application. An unassigned user may see `AADSTS50105` during sign-in.

> **Security note:** Prefer a named, accountable identity rather than a shared account when your organizational policies allow it. This desktop flow requires a user to complete interactive sign-in and any required MFA; it does not support unattended client-secret or service-principal authentication. Group-based assignments can have Entra licensing prerequisites.

## 4. Give the technical account workspace-level Azure RBAC access

Assign the role to the **same Entra user account that will sign in** to the desktop app. Do not assign it to the app registration's client ID as a substitute for the user.

1. Open the [Azure portal](https://portal.azure.com/).
2. Navigate to **Log Analytics workspaces** and select the workspace SentinelOps Inspector will monitor.
3. Select **Access control (IAM) → Add → Add role assignment**.
4. Search for and select **Log Analytics Reader**.
5. On **Members**, select **User, group, or service principal**, then select the **technical user account**.
6. Review and assign the role.
7. Confirm that the **scope is the individual Log Analytics workspace resource**. Do not grant subscription-wide or resource-group-wide access unless your organization separately requires it.

| Setting | Value |
| --- | --- |
| Azure role | **Log Analytics Reader** |
| Assignee | **The Entra user account signing in** |
| Scope | **Log Analytics workspace resource** |
| Intended capability | Read workspace logs without workspace administration |

This is an Azure RBAC assignment, separate from the Entra Enterprise Application user assignment in Step 3. Azure RBAC changes may take time to propagate.

## 5. Record the three application identifiers

| Identifier | Azure / Entra location | Used for |
| --- | --- | --- |
| **Tenant ID** | Entra ID → Overview, or App registrations → Overview | Identifies the tenant that authenticates the user |
| **Client ID** | Entra ID → App registrations → SentinelOps Inspector → Overview | Identifies the desktop application registration |
| **Workspace ID** | Azure portal → Log Analytics workspace → Properties (Workspace ID) | Identifies the Log Analytics workspace to query |

**No client secret is required or supported for this interactive desktop sign-in flow.**

A Workspace ID is not the same thing as the Azure Resource ID. Use the **Workspace ID (GUID)** shown in your workspace's Properties page.

## 6. Configure SentinelOps Inspector and validate

1. Launch **SentinelOpsInspector.exe**.
2. Open **Workspaces → Add workspace** (or edit an existing workspace).
3. Enter the **Tenant ID**, **Application (client) ID**, and **Workspace ID** you recorded.
4. Give the connection a recognizable display name and save.
5. Select the workspace as active.
6. Run **Test active workspace** or an equivalent connection test.
7. When the browser opens, sign in using the **technical account assigned in Step 3 and granted RBAC access in Step 4**.
8. Complete any organization-required MFA or Conditional Access steps.
9. After a successful connection test, open **Ingestion**, **Analytics**, or **Infrastructure Health** to use the read-only diagnostics.

An empty workspace can return a successful connection test but no billable `Usage` rows. This is expected if you have not started ingesting logs; **zero ingestion is not the same as failed authentication**.

## Troubleshooting

| Symptom | What to verify |
| --- | --- |
| `AADSTS50105`: user not assigned | Enterprise applications → SentinelOps Inspector → **Users and groups**; check **Assignment required?** and whether the user is assigned |
| Consent required / `AADSTS65001` | Log Analytics API delegated `Data.Read` and whether admin consent is required |
| `AADSTS500011`: resource not found | Correct tenant and application configuration; authentication audience must match Log Analytics API rather than an unsupported custom resource |
| HTTP 403 / Authorization failure | Signed-in identity has **Log Analytics Reader** on the **workspace resource**; RBAC and consent have propagated |
| Connection timeout | Corporate proxy, VPN, DNS, firewall, and outbound HTTPS reachability to Microsoft identity and Azure Monitor API services |
| Connection succeeds but dashboards show no data | Verify actual log ingestion, selected UTC time range, and whether the query includes only billable usage |
| Browser sign-in appears unexpectedly | Persistent token cache availability, account change, MFA, Conditional Access, or tenant session policies |

## Configuration checklist

- [ ] Entra application registered as a desktop/public client
- [ ] `http://localhost` desktop redirect URI configured
- [ ] Public client flows enabled
- [ ] Log Analytics API delegated `Data.Read` permission added and consented if required
- [ ] Technical **user** assigned to the Enterprise Application (when assignment restriction is enabled)
- [ ] Same user given **Log Analytics Reader** at the **workspace resource** scope
- [ ] Tenant ID recorded
- [ ] Application (client) ID recorded
- [ ] Log Analytics Workspace ID recorded
- [ ] No client secret created or supplied
- [ ] Live connection test succeeds

## Official Microsoft documentation

- [Azure Monitor Logs API access and authentication](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/api/access-api)
- [Register and configure a desktop application](https://learn.microsoft.com/en-us/azure/active-directory/develop/scenario-desktop-app-registration)
- [Assign a user to an Enterprise Application](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/assign-user-or-group-access-portal)
- [Require application user assignment](https://learn.microsoft.com/en-us/entra/identity-platform/howto-restrict-your-app-to-a-set-of-users)
- [Manage Log Analytics workspace access](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/manage-access)

**SentinelOps Inspector is an independent third-party application and is not affiliated with or endorsed by Microsoft.**
