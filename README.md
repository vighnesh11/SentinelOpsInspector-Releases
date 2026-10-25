<div align="center">
  <img src="assets/sentinelops-icon.png" alt="SentinelOps Inspector shield" width="105" />

  # SentinelOps Inspector

  **Windows desktop application for Microsoft Sentinel operational visibility**

  Read-only workspace diagnostics · Ingestion analysis · Data source health · KQL analytics
</div>

---

## Download for Windows

**[Download SentinelOps Inspector for Windows](https://github.com/vighnesh11/SentinelOpsInspector-Releases/releases)**

Go to the release's **Assets** section and download `SentinelOpsInspector.exe`. Save it to your Windows PC, then run it.

> **Beta preview:** SentinelOps Inspector is under active development. Test it in a non-production or explicitly authorized Azure environment before using it operationally. Do not rely on it as your sole monitoring or alerting solution.

### Requirements

- Windows 10 or 11 (64-bit).
- Internet access to the Microsoft Entra ID sign-in endpoints and Azure Monitor Logs API.
- Access to an Azure Log Analytics workspace.
- A suitable Microsoft Entra ID public-client app registration with **Log Analytics API** delegated **Data.Read**, and a user account with appropriate workspace read permissions.

### Azure and Microsoft Entra ID integration guide

**[Step-by-step integration guide: App registration, delegated Data.Read, user assignment, and workspace RBAC](INTEGRATION_GUIDE.md)**

Before signing in, follow the guide to configure a desktop public-client app registration, assign the designated Entra user to the enterprise application when assignment is required, and grant that same user **Log Analytics Reader** on the target workspace resource. Record the **Tenant ID**, **Client ID**, and **Workspace ID**. **No client secret is needed.**

### First-time configuration

1. Start `SentinelOpsInspector.exe`.
2. Open **Workspaces** and configure the tenant ID, application (client) ID, and Log Analytics workspace ID for your own Azure environment.
3. Sign in with your Microsoft work account when prompted.
4. Test the workspace connection, then open **Ingestion**, **Infrastructure Health**, or **Analytics**.

**For administrators:** Create a dedicated Microsoft Entra application registration in your tenant, enable the mobile/desktop public client redirect URI `http://localhost`, and grant the required delegated API permission. The signed-in account also needs Log Analytics workspace read access. Your own organization's consent and Conditional Access policies may apply. **Do not enter client secrets.**

A newly created or empty workspace can return zero usage results without indicating that authentication failed. Some functions require actual ingested data.

## What is included

- Workspace profiles and Microsoft Entra sign-in.
- Ingestion trends and explainable anomaly findings.
- Data source freshness checks.
- KQL query tools and diagnostics.
- PDF and CSV report exports.
- Custom branded desktop interface with dark/light themes.

**Not yet available as verified production features:** complete Sentinel detection-rule coverage, authoritative Azure billing attribution, AI-generated investigations, and automated remediation.

## Download safety and integrity

Only download releases from this repository's **Releases** section. Check the SHA-256 hash supplied in the release notes against your local download. Windows may show warnings for unsigned beta builds; verify provenance rather than bypassing your organization's security requirements.

The application is intended to run with *delegated, read-only permissions*. Keep workspace access scoped to least privilege.

## Source code and license

This is the **public binary-distribution repository**, not the development source repository. GitHub's automatic “Source code (zip/tar.gz)” links contain only the public repository documentation and assets, **not** the Python application source.

This repository does not grant an open-source license to the application. SentinelOps Inspector is an independently developed product and is not affiliated with or endorsed by Microsoft.

## Feedback

For beta issues, open a GitHub Issue on this repository. Never include access tokens, secrets, customer data or full internal identifiers in public issues.
