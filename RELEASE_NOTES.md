# SentinelOps Inspector v0.3A.2 — Windows Beta

**Status:** Beta / developer preview  
**Platform:** Windows 10/11 x64  
**Distribution:** Single-file `SentinelOpsInspector.exe` (attached separately as a GitHub Release asset)

## Highlights

- Native-feeling Windows desktop shell with navigation, workspace selector, dark/light themes and custom branding.
- Microsoft Entra interactive authentication with local token-cache persistence.
- Read-only Azure Monitor Logs queries, ingestion insights and data-source freshness inspection.
- KQL tools and exportable PDF/CSV reporting.

## Setup

1. Download the EXE from **Assets**.
2. Run it on an authorized Windows PC.
3. Supply your own tenant ID, app registration client ID and workspace ID in **Workspaces**.
4. Sign in with an account authorized to read the Log Analytics workspace.

## Limitations

- Some product areas remain preview or unavailable; this is not a full Sentinel replacement.
- Ingestion insights require logs; empty workspaces can show no data.
- Initial application setup requires an appropriate Entra public-client app registration and workspace permissions.
- This release is not yet code-signed or backed by a managed installer.

## SHA-256 checksum

After building the binary on the distribution PC, obtain its exact SHA-256 hash using PowerShell:

```powershell
Get-FileHash -Path .\dist\SentinelOpsInspector.exe -Algorithm SHA256
```

**Paste the resulting SHA-256 value into this section of your GitHub release description before publishing.** Do not publish a fabricated or placeholder checksum as if it were real.

## Feedback

Report non-sensitive problems through this repository's Issues section. Do not post confidential workspace details, tokens or production logs.
