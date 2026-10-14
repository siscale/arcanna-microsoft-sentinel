# Arcanna for Microsoft Sentinel

Deploy the Arcanna integration for Microsoft Sentinel in two steps.

## 1. Deploy Arcanna Custom Connector

The custom connector provides Microsoft Logic Apps access to the Arcanna API.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Ffeature%2Fguid-event-id-pipeline-tag%2FArcannaConnector%2Fazuredeploy.json)

During deployment provide:

- Arcanna protocol: `https` or `http`
- Arcanna host / IP, including port if required
- Custom connector name, normally `arcanna`

---

## 2. Deploy Arcanna Microsoft Sentinel Solution

This installs the Arcanna solution content into an existing Microsoft Sentinel workspace.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Ffeature%2Fguid-event-id-pipeline-tag%2FArcannaSentinelSolution%2Fazuredeploy.json)

During deployment provide:

- Microsoft Sentinel / Log Analytics workspace name
- Workspace Azure region

---

## After deployment

Open Microsoft Sentinel and go to:

**Automation → Playbook templates**

Create:

- `Arcanna-Sentinel-Triage`
- `Arcanna-Sentinel-Feedback`

For each playbook provide the required parameters, including:

- Azure region
- Arcanna custom connector name, normally `arcanna`
- Arcanna API key
- Arcanna Pipeline ID (triage only): the pipeline new incidents are sent to by default
- Microsoft Sentinel workspace name

The triage playbook tags each incident with `ArcannaPipeline:<id>` and the feedback playbook sends feedback to that pipeline, so you can change or route pipelines in the triage **Send Event** step without touching feedback. Arcanna Event IDs are Sentinel incident GUIDs.

For multiple Sentinel workspaces, repeat the installation per workspace and use a separate resource group for each workspace's connector and playbooks.

## Deployment order

1. Deploy the Arcanna Custom Connector
2. Deploy the Arcanna Microsoft Sentinel Solution
3. Create the Triage playbook
4. Create the Feedback playbook
5. Verify the generated API connections and automation rules

## Support

Arcanna.ai  
support@arcanna.ai  
https://arcanna.ai/contact
