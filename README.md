# Arcanna for Microsoft Sentinel

Send Sentinel incidents and related alerts to Arcanna for triage. The playbooks
write Arcanna's decision back to the incident and return analyst feedback to Arcanna.

| Setup order | Start here |
| --- | --- |
| 1. Prepare Arcanna | [API key, integration, and pipeline](#preparation) |
| 2. [Deploy the Arcanna Custom Connector](#arcanna-custom-connector) | [![Deploy the Arcanna Custom Connector to Azure](https://aka.ms/deploytoazurebutton)][deploy-connector] |
| 3. [Deploy the Arcanna Microsoft Sentinel solution](#arcanna-microsoft-sentinel-solution) | [![Deploy the Arcanna Sentinel solution to Azure](https://aka.ms/deploytoazurebutton)][deploy-solution] |
| 4. Grant playbook permissions | [Defender and Azure portal paths](#grant-playbook-permissions) |
| 5. Create the playbooks | [Triage first, then feedback](#create-the-playbooks) |
| 6. Verify the deployment | [Resource checks and a test incident](#post-deployment-checks-and-modifications) |

> **Microsoft Sentinel portal change**
>
> Microsoft supports Sentinel in the Azure portal through **March 31, 2027**.
> After that date, Sentinel users will be redirected to the **Microsoft Defender portal**.
> Use the same two deployment buttons for either portal.
> [Microsoft's transition notice][portal-notice]

[Contact Arcanna support][arcanna-contact] · [support@arcanna.ai](mailto:support@arcanna.ai)

## Preparation

- Confirm Azure can reach your running Arcanna instance.
- Identify the Azure subscription and target resource group.
- Have the name and region of an existing Log Analytics workspace with Sentinel enabled.

### Prepare Arcanna

1. **Create an API key** under **Platform Settings → API Keys → Create API Key**.
   - Copy and save the key when you create it.
2. **Create the integration** under **Integrations → Azure Sentinel → Create Integration**.
   - Assign the saved API key to the integration.
3. **Create a pipeline** and add the Azure Sentinel integration as an input.
   - Copy the **Pipeline ID / Job ID** from the pipeline URL or the first column
     of the pipelines table.

## Deployment

### Arcanna Custom Connector

Use the first **Deploy to Azure** button. The connector gives Logic Apps access
to the Arcanna API.

- **Protocol** — Select `https` or `http` to match the API.
- **Host** — Enter only the hostname or IP. Append a custom port if required
  (the standard HTTPS port is `443`).
  - With protocol `http` and host `10.20.30.40`, the connector builds
    `http://10.20.30.40/api/v1`.
- **Custom connector name** — `arcanna` is recommended.

### Arcanna Microsoft Sentinel solution

Use the second **Deploy to Azure** button to install the playbook templates
in your workspace.

- **Workspace** — Enter the exact Log Analytics workspace name, for example `arcanna-sentinel`.
- **Workspace location** — Enter that workspace's Azure region, for example `eastus`.
- **Location** — Keep the default unless your Azure environment requires another value.

### Grant playbook permissions

1. Open **Playbook permissions** for the target workspace.
   - **Defender portal** — Open **Settings → Microsoft Sentinel**, select the
     workspace, then **Playbook permissions**.
   - **Azure portal** — Open **Microsoft Sentinel**, select the workspace, then
     **Settings → Settings tab → Playbook permissions**.
2. Select **Configure permissions**.
3. Select the resource group where the Arcanna playbooks will be deployed, then
   select **Apply**. [Microsoft's permissions guide][permissions-guide]

### Create the playbooks

Open **Playbook templates** in the portal your workspace uses.

- **Defender portal** — **Microsoft Sentinel → Configuration → Automation → Playbook templates**.
- **Azure portal** — From **Microsoft Sentinel**, select your workspace, then
  **Configuration → Automation → Playbook templates**.

Use the same values for both playbooks.

| Setting | Value |
| --- | --- |
| Custom connector name | The name you deployed in step 2. |
| Arcanna API key | The key assigned to the Azure Sentinel integration in Arcanna. |
| Sentinel workspace name | The exact Log Analytics workspace name. |
| Location | The Azure region for the Logic Apps and their API connections. |

1. Select `Arcanna - Triage Microsoft Sentinel incidents` and choose **Create playbook**.
   - Set **Arcanna Pipeline ID** to your saved **Pipeline ID / Job ID**.
   - The recommended playbook name is `Arcanna-Sentinel-Triage`.
2. Once triage is deployed, select `Arcanna - Send analyst feedback` and choose
   **Create playbook**. [Microsoft's template guide][template-guide]

## Post Deployment checks and modifications

### Check the deployment

- [ ] **Arcanna** — The pipeline uses the Azure Sentinel integration as an input.
      The integration and playbooks use the same API key.
- [ ] **Azure** — The custom connector and both Logic Apps exist.
      The generated API connections are healthy.
- [ ] **Sentinel permissions** — Sentinel can run playbooks in the target resource group.
- [ ] **Sentinel automation** — Under **Automation → Automation rules**, the rule
      is enabled and its **Run playbook** action points to the triage playbook.

### Test an incident

1. Create a Sentinel test incident that matches the triage automation rule.
2. In Arcanna, confirm the incident and its related alerts reach the expected pipeline.
   - Arcanna event IDs are based on Sentinel incident GUIDs.
3. After inference completes, open the test incident and check for Arcanna's decision
   and result details.
   - **Defender portal** — **Incidents & alerts → Incidents**.
   - **Azure portal** — **Microsoft Sentinel → your workspace → Incidents**.
4. Close or classify the test incident and check that feedback reaches the same pipeline.
   - Triage adds the `ArcannaPipeline:<id>` tag. `Arcanna-Sentinel-Feedback`
     reads that tag and sends feedback to the selected pipeline through the custom connector.
   - Feedback labels must match the pipeline's configured classes, for example
     `Drop` or `Escalate`.

### Change pipeline routing

- To change the default destination, edit the Pipeline ID in the triage Logic App's
  **Send Event** action.
- To route incidents to different pipelines, choose the ID by incident source,
  severity, alert product, tenant, customer, or another incident field.

### Add another Sentinel workspace

- Repeat the deployment for each workspace.
- A separate resource group per workspace is recommended for its connector,
  Logic Apps, API connections, and automation rules.

[deploy-connector]: https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaConnector%2Fazuredeploy.json
[deploy-solution]: https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaSentinelSolution%2Fazuredeploy.json
[portal-notice]: https://learn.microsoft.com/en-us/azure/sentinel/microsoft-sentinel-defender-portal
[permissions-guide]: https://learn.microsoft.com/en-us/azure/sentinel/automation/run-playbooks
[template-guide]: https://learn.microsoft.com/en-us/azure/sentinel/automation/use-playbook-templates
[arcanna-contact]: https://arcanna.ai/contact
