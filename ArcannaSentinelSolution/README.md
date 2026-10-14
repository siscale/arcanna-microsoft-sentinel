# Arcanna for Microsoft Sentinel

This template deploys the Arcanna Microsoft Sentinel solution content into an existing Microsoft Sentinel workspace.

## Prerequisite

Deploy the Arcanna Custom Connector first:

[Deploy Arcanna Custom Connector](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaConnector%2Fazuredeploy.json)

## Deploy Arcanna Sentinel Solution

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaSentinelSolution%2Fazuredeploy.json)

During deployment provide:

- `workspace` — Microsoft Sentinel / Log Analytics workspace name
- `workspace-location` — Azure region of the workspace

After deployment:

1. Open Microsoft Sentinel
2. Go to **Content hub**
3. Open **Arcanna**
4. Create the **Arcanna-Sentinel-Triage** playbook
5. Create the **Arcanna-Sentinel-Feedback** playbook

For playbook creation provide:

- `location` — Azure region, for example `eastus`
- `ArcannaCustomConnectorName` — normally `arcanna`
- `ArcannaApiKey` — Arcanna API key
- `ArcannaPipelineId` — Arcanna pipeline/job ID
- `SentinelWorkspaceName` — Microsoft Sentinel workspace name
