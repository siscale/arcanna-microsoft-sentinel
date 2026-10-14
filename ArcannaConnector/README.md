# Arcanna Custom Connector for Microsoft Sentinel

This template deploys the Arcanna Custom Azure Logic Apps Connector required by the Arcanna Microsoft Sentinel playbooks.

## Deploy to Azure

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Ffeature%2Fguid-event-id-pipeline-tag%2FArcannaConnector%2Fazuredeploy.json)

## Parameters

- `location` - Azure region for the connector.
- `CustomConnectorName` - Connector name. Default: `arcanna`.
- `ArcannaBaseUrl` - Arcanna endpoint origin including scheme and port if required. Do not include `/api/v1`.

Example:

`https://arcanna.example.com`

or:

`https://arcanna.example.com:9666`

## After deployment

Return to Microsoft Sentinel:

`Automation -> Playbook templates`

Then create the Arcanna Triage or Feedback playbook.
