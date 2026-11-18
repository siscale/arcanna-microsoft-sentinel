# Arcanna for Microsoft Sentinel

This repository contains the deployment templates required to integrate Arcanna with Microsoft Sentinel.

The setup has three parts:

1. Prepare Arcanna
2. Deploy the Arcanna Custom Connector
3. Deploy the Microsoft Sentinel solution and create the playbooks

Once configured, Microsoft Sentinel incidents can be sent to Arcanna for triage, and analyst feedback from Sentinel can be sent back to Arcanna.

---

## 1. Prepare Arcanna

Before deploying the Microsoft Sentinel integration, make sure Arcanna is already running and reachable from Azure.

You will need to configure three things in Arcanna:

1. An API key
2. An Azure Sentinel integration
3. A pipeline that uses that integration as an input

### 1.1 Create an API key

In Arcanna:

`Platform Settings → API Keys → Create API Key`

Create a new API key for the Microsoft Sentinel integration.

**Important:** copy and save the API key when it is created.

You will need this value later when creating the Sentinel playbooks.

### 1.2 Create an Azure Sentinel integration

In Arcanna:

`Platform Settings → Integrations → Create Integration → Azure Sentinel`

Create the Azure Sentinel integration and assign the API key created in the previous step.

The integration must use the same API key that will later be configured in the Microsoft Sentinel playbooks.

### 1.3 Create an Arcanna pipeline

Create a new Arcanna pipeline and configure the previously created Azure Sentinel integration as an input.

After creating the pipeline, note its **Pipeline ID / Job ID**.

You will need this value when creating the Microsoft Sentinel triage playbook.

Before continuing, you should have:

- Arcanna hostname or IP
- Arcanna API key
- Arcanna Azure Sentinel integration
- Arcanna Pipeline ID / Job ID

---

## 2. Deploy Arcanna Custom Connector

The custom connector provides Microsoft Logic Apps access to the Arcanna API.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaConnector%2Fazuredeploy.json)

During deployment provide:

- Arcanna protocol: `https` or `http`
- Arcanna host / IP, including port if required
- Custom connector name, normally `arcanna`

### Arcanna protocol

Select either:

- `https`
- `http`

depending on how the Arcanna API is exposed.

### Arcanna Host

Enter only the hostname or IP address.

Do **not** include:

- `https://`
- `http://`
- `/api/v1`
- any individual API endpoint

If Arcanna uses a non-standard port, include the port.

Examples:

```text
arcanna.example.com
```

```text
arcanna.example.com:8443
```

```text
10.20.30.40:9666
```

For example:

```text
Protocol = http
Host     = 10.20.30.40:9666
```

The connector will build the Arcanna API URL automatically:

```text
http://10.20.30.40:9666/api/v1
```

### Custom Connector Name

The recommended value is:

```text
arcanna
```

Unless you have a specific reason to change it, keep the default.

---

## 3. Deploy Arcanna Microsoft Sentinel Solution

This installs the Arcanna solution content into an existing Microsoft Sentinel workspace.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsiscale%2Farcanna-microsoft-sentinel%2Fmain%2FArcannaSentinelSolution%2Fazuredeploy.json)

This deployment installs the Arcanna content templates into Sentinel.

It does **not** create the final Logic Apps yet.

During deployment provide:

- Microsoft Sentinel / Log Analytics workspace name
- Workspace Azure region

### Workspace

Enter the exact name of the Log Analytics workspace that has Microsoft Sentinel enabled.

Example:

```text
arcanna-sentinel
```

### Workspace location

Enter the Azure region of that workspace.

Example:

```text
eastus
```

The `Location` field can remain at its default value unless your Azure environment requires otherwise.

---

## 4. Grant Microsoft Sentinel permission to run playbooks

Before creating the playbooks, Microsoft Sentinel must have permission to execute Logic Apps from the resource group where the playbooks will be deployed.

In the Microsoft Defender portal:

`Microsoft Defender → Settings → Microsoft Sentinel → Select the workspace → Playbook permissions`

Then:

1. Open **Configure permissions**
2. Select the resource group where the Arcanna playbooks will be deployed
3. Click **Grant permissions**

This gives Microsoft Sentinel the required permission to execute playbooks from that resource group.

Do this before creating the playbooks.

---

## 5. Create the Arcanna playbooks

After the Sentinel solution deployment succeeds, open Microsoft Sentinel automation.

In the Microsoft Defender portal:

`Microsoft Defender → Microsoft Sentinel → Automation → Playbook templates`

Depending on the current Microsoft UI, the exact navigation may vary slightly.

You should see the Arcanna templates:

- `Arcanna - Triage Microsoft Sentinel incidents`
- `Arcanna - Send analyst feedback`

Create the triage playbook first.

---

## 6. Create the Triage playbook

Select:

`Arcanna - Triage Microsoft Sentinel incidents`

and create the playbook.

Provide the required parameters.

### Playbook Name

Recommended:

```text
Arcanna-Sentinel-Triage
```

### Arcanna Custom Connector Name

If you used the recommended connector name:

```text
arcanna
```

### Arcanna API Key

Enter the API key created earlier in Arcanna.

This must be the same API key assigned to the Azure Sentinel integration in Arcanna.

### Arcanna Pipeline ID

Enter the **Pipeline ID / Job ID** of the Arcanna pipeline created earlier.

This is the pipeline that new Microsoft Sentinel incidents will be sent to by default.

### Sentinel Workspace Name

Enter the exact Microsoft Sentinel / Log Analytics workspace name.

Example:

```text
arcanna-sentinel
```

### Location

Enter the Azure region where the Logic App and its API connections should be deployed.

Example:

```text
eastus
```

---

## 7. Create the Feedback playbook

After the triage playbook has been created successfully, create:

`Arcanna - Send analyst feedback`

Use the same:

- Arcanna custom connector
- Arcanna API key
- Sentinel workspace
- Azure region

The feedback playbook sends analyst feedback from Microsoft Sentinel back to Arcanna.

The triage playbook tags each incident with:

```text
ArcannaPipeline:<id>
```

The feedback playbook uses that value to send feedback to the correct Arcanna pipeline.

This means the triage Logic App can later be modified to route incidents to different Arcanna pipelines without requiring corresponding changes to the feedback Logic App.

Arcanna Event IDs are based on Microsoft Sentinel incident GUIDs.

---

## 8. How the triage flow works

The runtime flow is:

```text
Microsoft Sentinel Incident
        │
        ▼
Automation Rule
        │
        ▼
Arcanna-Sentinel-Triage
        │
        ▼
Arcanna Custom Connector
        │
        ▼
Arcanna Pipeline
        │
        ▼
Arcanna Decision
        │
        ▼
Microsoft Sentinel
```

The triage playbook:

1. Receives a Microsoft Sentinel incident
2. Sends the incident and related alerts to Arcanna
3. Uses the configured Arcanna Pipeline ID as the default destination
4. Waits for Arcanna inference to complete
5. Retrieves the Arcanna result
6. Adds the Arcanna decision to the Sentinel incident
7. Updates the incident with Arcanna result information

---

## 9. Changing the Arcanna pipeline after deployment

The Pipeline ID configured during playbook creation is the default pipeline used by the triage Logic App.

It can be changed later.

After deployment, you can edit the Logic App and modify the Pipeline ID passed to the Arcanna **Send Event** action.

For example, instead of always sending incidents to one pipeline, you can implement routing based on:

- incident source
- severity
- alert product
- tenant
- customer
- incident fields
- other Logic App conditions

The Pipeline ID configured during setup simply provides a working default so the integration works immediately after deployment.

---

## 10. Analyst feedback flow

The feedback flow works in the opposite direction:

```text
Microsoft Sentinel Analyst
        │
        ▼
Incident closed / classified
        │
        ▼
Arcanna-Sentinel-Feedback
        │
        ▼
Arcanna Custom Connector
        │
        ▼
Arcanna Feedback API
```

This allows analyst decisions in Microsoft Sentinel to be returned to Arcanna.

Typical Arcanna feedback labels include:

- `Drop`
- `Escalate`

The exact values should match the feedback classes configured on the Arcanna pipeline.

---

## 11. Recommended deployment order

For a new environment, use the following order:

1. Deploy Arcanna
2. Create an Arcanna API key
3. **Save the API key**
4. Create an Azure Sentinel integration in Arcanna
5. Assign the API key to the integration
6. Create an Arcanna pipeline
7. Assign the Azure Sentinel integration as an input to the pipeline
8. Copy the Pipeline ID / Job ID
9. Deploy the Arcanna Custom Connector in Azure
10. Deploy the Arcanna Microsoft Sentinel Solution
11. Grant Microsoft Sentinel playbook permissions for the target resource group
12. Create the Triage playbook
13. Create the Feedback playbook
14. Verify the generated API connections and automation rules
15. Test with a Microsoft Sentinel incident

---

## 12. What you need before starting

Prepare the following values before deploying:

| Value | Where to get it |
|---|---|
| Arcanna hostname or IP | Arcanna deployment |
| Arcanna port | Arcanna deployment, if non-standard |
| Arcanna protocol | `https` or `http` |
| Arcanna API key | Arcanna Platform Settings |
| Arcanna Pipeline ID / Job ID | Arcanna pipeline |
| Azure subscription | Azure |
| Resource group | Azure |
| Sentinel workspace name | Microsoft Sentinel / Log Analytics |
| Sentinel workspace region | Azure |
| Custom connector name | Recommended: `arcanna` |

---

## 13. Verification

After deployment, verify the following.

### Arcanna

Confirm that:

- the Azure Sentinel integration exists
- the integration uses the correct API key
- the Arcanna pipeline has the Azure Sentinel integration configured as an input

### Azure

Confirm that the Arcanna custom connector exists.

Confirm that the Logic Apps exist:

```text
Arcanna-Sentinel-Triage
Arcanna-Sentinel-Feedback
```

Confirm that the generated API connections are healthy.

### Microsoft Sentinel

Confirm that:

- the automation rule is enabled
- the triage playbook is associated with the automation rule
- Microsoft Sentinel has playbook permissions on the resource group

Create a test incident and confirm the full flow:

```text
Sentinel incident
→ Arcanna event
→ Arcanna inference
→ Sentinel update
```

---

## 14. Multiple pipelines

A single deployment initially uses one Arcanna Pipeline ID as the default.

If more advanced routing is required, modify the triage Logic App after deployment.

The **Send Event** action can use:

- a fixed Pipeline ID
- a dynamically calculated Pipeline ID

This allows one Sentinel integration to route different incidents to different Arcanna pipelines.

Because the triage playbook stores the selected pipeline on the incident using the `ArcannaPipeline:<id>` tag, the feedback playbook can still return analyst feedback to the correct pipeline.

---

## 15. Multiple Sentinel workspaces

For multiple Microsoft Sentinel workspaces, repeat the installation per workspace.

It is recommended to use a separate resource group for each workspace's:

- Arcanna custom connector
- Logic Apps
- API connections
- automation rules

This keeps permissions and deployment lifecycle isolated between Sentinel environments.

---

## Support

Arcanna.ai  
support@arcanna.ai  
https://arcanna.ai/contact
