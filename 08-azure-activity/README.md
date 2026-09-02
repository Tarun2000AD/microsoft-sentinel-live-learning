<div align="center">

# 📥 Step 08 · Connect Azure Activity

### *The control-plane log — who did what to your Azure resources*

[![Phase](https://img.shields.io/badge/Phase-Data onboarding-1F6FEB?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~15 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-usually free-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

`AzureActivity` rows are flowing into the workspace, and you can see your own recent portal actions
in KQL.

## 🧠 Why this step

Azure Activity is the audit log of the Azure Resource Manager control plane: role assignments,
resource creation/deletion, NSG changes, Key Vault access-policy edits, deallocations. It is small,
high-signal, and typically **free to ingest**. It's the right first connector — it starts logging
your *own* lab-building actions immediately, so you have data to query in minutes.

## ✅ Prerequisites

- [Step 07](../07-connectors-and-content-hub/README.md) — Azure Activity solution installed
- Owner/Contributor on the subscription (to create the diagnostic setting)

## 🧭 Concepts in 60 seconds

- Modern Azure Activity ingestion is a **diagnostic setting on the subscription** that routes the
  Activity log to your workspace (the old "legacy" agent-style connector is retired).
- It lands in the **`AzureActivity`** table.
- Key columns: `OperationNameValue`, `Caller`, `CallerIpAddress`, `ActivityStatusValue`,
  `ResourceGroup`, `_ResourceId`, `Authorization` (contains the role/scope for RBAC changes).

## 🖱️ Do it — portal

1. **Microsoft Sentinel → Data connectors → Azure Activity → Open connector page.**
2. Under **Configuration**, click **Launch Azure Policy Assignment wizard** *or* the direct
   **Configure Azure Activity logs** link.
3. Simplest for a lab: **Add data connector via diagnostic settings** → pick the **subscription**
   `sub-sentinel-lab` → destination **Send to Log Analytics workspace** `law-sentinel-lab` → select
   log categories **Administrative, Security, Alert, Policy, Recommendation, ServiceHealth,
   ResourceHealth, Autoscale** → **Save**.

## 💻 Do it — CLI

```bash
SUB=$(az account show --query id -o tsv)
WS=$(az monitor log-analytics workspace show -g rg-sentinel-lab -n law-sentinel-lab --query id -o tsv)

az monitor diagnostic-settings subscription create \
  --name "activity-to-sentinel" \
  --location eastus \
  --workspace "$WS" \
  --logs '[
    {"category":"Administrative","enabled":true},
    {"category":"Security","enabled":true},
    {"category":"Alert","enabled":true},
    {"category":"Policy","enabled":true},
    {"category":"ServiceHealth","enabled":true},
    {"category":"ResourceHealth","enabled":true}
  ]'
```

## 🧪 Validate

Wait ~10–15 minutes, then in **Logs**:

```kusto
AzureActivity
| where TimeGenerated > ago(1h)
| summarize Events = count() by OperationNameValue, ActivityStatusValue
| sort by Events desc
```

```kusto
// your own footprint building the lab
AzureActivity
| where TimeGenerated > ago(1d)
| where Caller has "@"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup, CallerIpAddress
| sort by TimeGenerated desc
| take 20
```

**You should see** your recent `Microsoft.OperationalInsights/workspaces/write`,
`Microsoft.Authorization/roleAssignments/write` (from step 05) and similar operations, with your
UPN as `Caller`. If the table 404s, the diagnostic setting didn't save or you're too early — give
it 20 minutes.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Looking for the retired agent-based connector | Activity is a subscription diagnostic setting now |
| Setting it on the resource group | Activity log is a **subscription**-level source |
| Expecting instant data | First rows take 10–20 minutes |
| Ignoring `ServiceHealth`/`ResourceHealth` noise | Filter it in rules; it's still cheap to keep |

## 🗒️ Log your run

`LOG.md` — the diagnostic setting name, and the first `AzureActivity` query output (Caller/IP
redacted).

## 📚 Microsoft Learn

- [Connect Azure Activity logs to Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/data-connectors/azure-activity)
- [Azure Activity log schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/azureactivity)

---

<div align="center">
<sub>

[⬅ Prev: 07 · Connectors & Content hub](../07-connectors-and-content-hub/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 09 · Microsoft Entra ID ➡](../09-microsoft-entra-id/README.md)

</sub>
</div>
