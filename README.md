# Arcanna for Microsoft Sentinel

Public deployment assets for the Arcanna integration with Microsoft Sentinel.

This repository contains the Azure deployment template for the Arcanna Custom Azure Logic Apps Connector used by the Arcanna Microsoft Sentinel playbooks.

## Arcanna Custom Connector

Deploy the custom connector before creating the Arcanna playbooks in Microsoft Sentinel.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaConnector%2Fazuredeploy.json)

Deployment template:

`ArcannaConnector/azuredeploy.json`

Documentation:

`ArcannaConnector/README.md`

## Deployment order

1. Deploy the Arcanna Custom Connector.
2. Open Microsoft Sentinel.
3. Go to `Automation -> Playbook templates`.
4. Create the Arcanna Triage and/or Feedback playbook.
5. Configure the Arcanna API key and pipeline ID during playbook deployment.
6. Grant Microsoft Sentinel permission to run the playbooks where required.

## Support

Arcanna.ai

support@arcanna.ai

https://arcanna.ai/contact
