<div align="center">

# 🧱 Step 01 · Log Analytics workspace

### *Create the workspace Sentinel is built on top of*

[![Phase](https://img.shields.io/badge/Phase-Foundations-8661C5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~15 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240 idle-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

A single Log Analytics workspace in `rg-sentinel-lab`, in your chosen region, with a retention and
pricing tier you chose on purpose.

## 🧠 Why this step

Microsoft Sentinel is not a separate data store. It is a **solution installed onto a Log Analytics
workspace**. The workspace is where every table lives, where KQL runs, and what you pay ingestion
and retention on. Its region, name and retention are effectively permanent — the workspace can't be
moved regions, and renaming means rebuilding.

## ✅ Prerequisites

- [Step 00](../00-azure-subscription-and-tenant/README.md) — subscription, resource group, CLI signed in

## 🧭 Concepts in 60 seconds

- **One workspace** is right for almost every single-tenant lab and most small orgs. Step 53 covers
  when you'd want more.
- **Retention**: first **90 days are free** when Sentinel is enabled on the workspace. Beyond that
  you pay per GB/month. Set interactive retention to `90` for the lab.
- **Pricing tier**: leave it at **Pay-As-You-Go** for now. Commitment tiers (step 56) only pay off
  above ~100 GB/day.
- **Workspace ID vs resource ID**: the GUID you see is the *workspace ID* (used by agents). The ARM
  *resource ID* is `/subscriptions/.../workspaces/<name>`. Both matter later.

## 🖱️ Do it — portal

1. [portal.azure.com](https://portal.azure.com) → search **Log Analytics workspaces** → **Create**.
2. Resource group `rg-sentinel-lab`, name `law-sentinel-lab`, region = your step-00 region.
3. **Review + Create** → **Create**. Default tier is Pay-As-You-Go.
4. Once deployed: open it → **Settings → Usage and estimated costs → Data Retention** → set to
   **90 days** → **OK**.

## 💻 Do it — CLI / IaC

```bash
LOCATION=eastus
az monitor log-analytics workspace create \
  --resource-group rg-sentinel-lab \
  --workspace-name law-sentinel-lab \
  --location $LOCATION \
  --retention-time 90 \
  --sku PerGB2018 \
  --tags env=lab purpose=sentinel-live-learning
```

<details><summary>Bicep</summary>

```bicep
resource law 'Microsoft.OperationalInsights/workspaces@2023-09-01' = {
  name: 'law-sentinel-lab'
  location: resourceGroup().location
  properties: {
    retentionInDays: 90
    sku: { name: 'PerGB2018' }
    features: { enableLogAccessUsingOnlyResourcePermissions: true }
  }
}
output workspaceId string = law.id
```
</details>

## 🧪 Validate

```bash
az monitor log-analytics workspace show \
  -g rg-sentinel-lab -n law-sentinel-lab \
  --query "{name:name, region:location, retention:retentionInDays, sku:sku.name, id:customerId}" -o table
```

**You should see** `retention: 90`, `sku: PerGB2018`, and a `customerId` GUID (the workspace ID).
The workspace bills **nothing** until data flows in — right now it is empty.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Workspace in a different region from your resources | Cross-region ingestion cost, and some connectors expect co-location |
| Leaving retention at the 30-day default | You lose incident history a month back; 90 is free with Sentinel |
| Creating several "test" workspaces | Sentinel content, cost and RBAC fragment across them — hard to undo |
| Picking a commitment tier now | You have no idea of your volume yet; PAYG until step 56 |

## 🗒️ Log your run

`LOG.md` — record the workspace name and region. Do **not** paste the workspace ID unredacted.

## 📚 Microsoft Learn

- [Log Analytics workspace overview](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)
- [Design a Log Analytics workspace architecture](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/workspace-design)
- [Microsoft Sentinel workspace prerequisites](https://learn.microsoft.com/en-us/azure/sentinel/prerequisites)

---

<div align="center">
<sub>

[⬅ Prev: 00 · Azure subscription & tenant](../00-azure-subscription-and-tenant/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 02 · Enable Sentinel ➡](../02-enable-sentinel/README.md)

</sub>
</div>
