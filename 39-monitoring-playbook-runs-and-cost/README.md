<div align="center">

# 🔄 Step 39 · Monitoring playbook runs & cost

### *Find the failed runs, and know what automation actually costs*

[![Phase](https://img.shields.io/badge/Phase-Automation-3FB950?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~25 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You have a view of every playbook's run success rate, an alert when a playbook starts failing, and a
rough monthly cost estimate for your automation.

## 🧠 Why this step

A failing response playbook is a silent containment gap. And Logic Apps bill **per action** — a
playbook with a big `For each` loop over entities, running on every incident, adds up. Both need
watching.

## ✅ Prerequisites

- [Step 15](../15-ingestion-health-and-validation/README.md) — `SentinelHealth` enabled
- Several playbooks with run history

## 🧭 Where run data lives

| Signal | Place |
|---|---|
| Per-playbook runs | Logic App → **Runs history** (Succeeded / Failed / Running) |
| Sentinel-triggered playbook health | `SentinelHealth` where `SentinelResourceType == "Playbook"` |
| Automation rule → playbook outcome | `SentinelHealth` `SentinelResourceType == "Automation Rule"`, `RecordId` links |
| Logic App metrics | Logic App → **Metrics**: *Runs Completed*, *Runs Failed*, *Billing Usage* |
| Cost | Cost Management → filter Meter = *Logic Apps* / *Standard/Consumption actions* |

## 🖱️ Do it — portal

1. Each playbook → **Runs history** → filter **Failed**. Open one failure, read the errored action.
2. **Cost Management → Cost analysis** → scope `rg-sentinel-lab` → filter **Service name = Logic
   Apps** → group by **Resource**. Note per-playbook cost.
3. **Monitor → Workbooks** → save the **"Playbooks health monitoring"** / **"Automation"** template
   workbook if present in your Content hub.

## 💻 Do it — KQL

```kusto
// playbook success rate, 7 days
SentinelHealth
| where TimeGenerated > ago(7d) and SentinelResourceType == "Playbook"
| summarize Runs = count(), Failures = countif(Status != "Success") by SentinelResourceName
| extend FailPct = round(100.0 * Failures / Runs, 1)
| sort by FailPct desc
```

```kusto
// recent failures with reason
SentinelHealth
| where TimeGenerated > ago(2d) and SentinelResourceType == "Playbook" and Status != "Success"
| project TimeGenerated, SentinelResourceName, Status, Description, tostring(ExtendedProperties)
```

```kusto
// automation rules that failed to run their playbook
SentinelHealth
| where TimeGenerated > ago(2d) and SentinelResourceType == "Automation Rule" and Status != "Success"
| project TimeGenerated, SentinelResourceName, Description
```

**Build the alert**: scheduled rule (every 1h, lookback 2h) on the failures query, threshold > 0,
title `OPS · Playbook failing`, routed to the ops queue (not the security queue), severity Medium.

## 🧪 Validate

1. Break a playbook (revoke its MI role, or point an HTTP action at a bad URL). Run it.
2. Confirm within ~2h: `SentinelHealth` shows the failure, the `OPS · Playbook failing` rule raises
   an incident, and **Runs history** shows Failed.
3. Estimate cost: `runs/day × actions/run × $per-action`. For a Consumption Logic App, standard
   actions are on the order of ~$0.000025 each — so a 15-action playbook on 200 incidents/day is
   roughly `200 × 15 × $0.000025 ≈ $0.075/day`. Confirm against Cost Management.

**You should see** the failure detected and alerted, plus a defensible monthly automation cost
number in `LOG.md`.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| No alert on playbook failure | Containment silently stops working |
| `For each` over unbounded entity lists | Action count (and cost, and rate limits) explode on a noisy incident |
| Ops alerts in the security queue | They bury real incidents |
| Retrying failed runs blindly | A poisoned input retries forever — cap retries |

## 🗒️ Log your run

`LOG.md` — the success-rate table, the failure-alert proof, and your automation cost estimate vs
actual.

## 📚 Microsoft Learn

- [Monitor the health and audit the integrity of your automation](https://learn.microsoft.com/en-us/azure/sentinel/monitor-automation-health)
- [Logic Apps pricing model](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-pricing)
- [Monitor run status and history in Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/monitor-logic-apps)

---

<div align="center">
<sub>

[⬅ Prev: 38 · Playbooks as code](../38-playbooks-as-code/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 40 · Hunting mindset & hypotheses ➡](../40-hunting-mindset-and-hypotheses/README.md)

</sub>
</div>
