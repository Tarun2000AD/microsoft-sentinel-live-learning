<div align="center">

# 🔄 Step 38 · Playbooks as code

### *ARM-template a playbook and redeploy it from scratch*

[![Phase](https://img.shields.io/badge/Phase-Automation-3FB950?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~40 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

`PB-Notify-Incident` exists as an ARM template in this repo, parameterised (no secrets, no
hard-coded connection IDs), and you can deploy it to a clean resource group and have it work.

## 🧠 Why this step

Playbooks are Logic Apps — `Microsoft.Logic/workflows` + `Microsoft.Web/connections`. A playbook
built only by clicking can't be reviewed or reproduced, and its ARM export is full of environment
IDs and (worse) callback URLs. This step teaches the clean export.

## ✅ Prerequisites

- [Step 30](../30-first-playbook-notify/README.md) — the playbook exists
- [Step 32](../32-playbook-managed-identity-and-permissions/README.md) — MI, so fewer connection secrets

## 🧭 What's in a playbook ARM template

| Resource | Notes |
|---|---|
| `Microsoft.Logic/workflows` | The definition (`$connections` param + `definition`) |
| `Microsoft.Web/connections` | One per connector (azuresentinel, teams, keyvault…). MI connections carry no secret; OAuth ones need post-deploy authorization |
| Parameters | workspace name/ID, connection names, Teams team/channel IDs, region |
| `identity` block | `type: SystemAssigned` |

**Never commit**: `accessEndpoint` / callback URLs, `parameterValues` with tokens, the workspace
GUID. The `.gitignore` blocks `*callback*`; the CI check (step 55 / `structure.yml`) greps for
`logic.azure.com/workflows/<hex>`.

## 🖱️ Do it — export cleanly

1. **Logic App → Export template** (left menu → Automation → Export template). Download.
2. **Better**: **Automation → Active playbooks → select → Export** (Sentinel's playbook export is
   tuned for this) — or use the community
   [`Playbook-ARM-Template-Generator`](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Playbook-ARM-Template-Generator).
3. In the JSON:
   - Replace literal workspace/connection IDs with `parameters(...)`.
   - Delete any `parameterValues` containing tokens; for OAuth connectors leave
     `"status": "Connected"` out and authorize post-deploy.
   - Add a `metadata` block so it shows as a Sentinel playbook in Content hub.
4. Save to `artifacts/playbooks/pb-notify-incident.json` + a `parameters.sample.json`.

## 💻 Do it — deploy to a clean RG

```bash
az group create -n rg-sentinel-lab-deploytest -l eastus

az deployment group create -g rg-sentinel-lab-deploytest \
  --template-file artifacts/playbooks/pb-notify-incident.json \
  --parameters @artifacts/playbooks/parameters.sample.json \
  --parameters workspaceName=law-sentinel-lab PlaybookName=PB-Notify-Incident-2

# authorize the OAuth connections (Teams/Office365) in the portal once:
# rg-sentinel-lab-deploytest -> API Connections -> each -> Edit API connection -> Authorize
```

## 🧪 Validate

```bash
az resource show -g rg-sentinel-lab-deploytest -n PB-Notify-Incident-2 \
  --resource-type Microsoft.Logic/workflows --query "{name:name, state:properties.state, mi:identity.type}" -o table
```

Attach `PB-Notify-Incident-2` to a test automation rule and fire an incident. Confirm the Teams
message arrives and the run is Succeeded.

Then **grep your template** before committing:

```bash
grep -nE 'logic.azure.com|sig=|accessKey|[0-9a-f]{32}' artifacts/playbooks/pb-notify-incident.json || echo "clean"
```

**You should see** the playbook deploy and run from the template alone (plus a one-time OAuth
authorize), and the grep returns `clean`. Delete `rg-sentinel-lab-deploytest` afterwards.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Committing the raw portal export | Callback URLs = anyone can trigger your playbook |
| Hard-coded connection resource IDs | Won't deploy to another RG/sub |
| Expecting OAuth connections to "just work" | Teams/Outlook need a manual authorize after deploy |
| No `parameters.sample.json` | Nobody else can deploy it |

## 🗒️ Log your run

`LOG.md` — the clean-grep result and the deploy-to-fresh-RG proof. Commit `artifacts/playbooks/`.

## 📚 Microsoft Learn

- [Export and deploy Microsoft Sentinel playbooks](https://learn.microsoft.com/en-us/azure/sentinel/use-playbook-templates)
- [Microsoft.Logic/workflows ARM reference](https://learn.microsoft.com/en-us/azure/templates/microsoft.logic/workflows)
- [Automate Logic Apps deployment with ARM templates](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-azure-resource-manager-templates-overview)

---

<div align="center">
<sub>

[⬅ Prev: 37 · Guardrails and conditions](../37-guardrails-and-conditions/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 39 · Monitoring playbook runs & cost ➡](../39-monitoring-playbook-runs-and-cost/README.md)

</sub>
</div>
