<div align="center">

# 📥 Step 15 · Ingestion health & validation

### *Prove the data is flowing — and catch the day it silently stops*

[![Phase](https://img.shields.io/badge/Phase-Data onboarding-1F6FEB?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You have a health workbook / saved queries that show, per source: last event time, volume trend,
and agent heartbeat — plus one analytics rule that alerts when a source goes quiet.

## 🧠 Why this step

A connector that stops is worse than one you never had, because the dashboards still look fine.
Causes: a rotated API key, a deleted diagnostic setting, an agent that fell off auto-upgrade, a VM
left deallocated, hitting a daily cap. You need to *monitor the monitoring*.

## ✅ Prerequisites

- Steps 08–14 — you have several sources connected
- [Step 07](../07-connectors-and-content-hub/README.md) — install the **SentinelHealth** / **Health & Audit** solution if not already

## 🧭 Where health lives

| Signal | Table / place |
|---|---|
| Agent alive | `Heartbeat` (one row/min per AMA machine) |
| Connector data received | per source table's newest `TimeGenerated` |
| Connector-level health events | `SentinelHealth` (enable in Settings → Auditing and health monitoring) |
| Daily cap hit | `Operation` where `Detail has "data collection"` / workspace **Usage and estimated costs** |
| Rule / automation health | `SentinelHealth` with `SentinelResourceType` = `Analytics Rule` / `Automation Rule` |

## 🖱️ Do it — portal

1. **Settings → Settings → Auditing and health monitoring → Enable.** This populates
   `SentinelHealth`.
2. **Workbooks → Templates → "Data collection health monitoring"** and **"Analytics efficiency"**
   → **Save** both.
3. **Workbooks → "Workspace usage report"** → save. This is your cost + volume view.

## 💻 Do it — saved queries / KQL

```kusto
// last event per key table — your "is it flowing?" board
union withsource=T
    (Heartbeat), (SigninLogs), (AzureActivity), (SecurityEvent), (Syslog),
    (CommonSecurityLog), (DeviceProcessEvents), (SecurityIncident)
| summarize LastEvent = max(TimeGenerated), Rows24h = countif(TimeGenerated > ago(24h)) by T
| extend StaleHours = round((now() - LastEvent) / 1h, 1)
| sort by StaleHours desc
```

```kusto
// agent heartbeat gaps
Heartbeat
| where TimeGenerated > ago(24h)
| summarize LastBeat = max(TimeGenerated) by Computer
| extend GapMinutes = round((now() - LastBeat)/1m, 0)
| where GapMinutes > 15
```

```kusto
// connector health failures
SentinelHealth
| where TimeGenerated > ago(7d)
| where Status != "Success"
| project TimeGenerated, SentinelResourceName, SentinelResourceType, Status, Description
```

## 🧪 Validate — build the "source went quiet" rule

**Analytics → Create → Scheduled query rule:**

```kusto
let expected = dynamic(["SigninLogs","AzureActivity","SecurityEvent"]);
union withsource=T (SigninLogs), (AzureActivity), (SecurityEvent)
| summarize LastEvent = max(TimeGenerated) by T
| where LastEvent < ago(2h)
| extend Reason = strcat(T, " has no events for ", round((now()-LastEvent)/1h,1), "h")
```

Run every 1h, lookback 4h. Then **stop a connector on purpose** (disable the Entra diagnostic
setting) and confirm the rule raises an incident within ~2 hours; re-enable it and close the
incident.

**You should see** the incident fire on the deliberately-broken source and the "last event" board
flag it before any dashboard would.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Trusting the Data connectors page's "Connected" badge | It reflects config, not recent data |
| No alert on silence | You find out when an investigation comes up empty |
| Monitoring only volume, not recency | A source can trickle a few stale rows and still be "broken" |
| Not enabling `SentinelHealth` | You lose the connector/rule/automation failure signal entirely |

## 🗒️ Log your run

`LOG.md` — the "last event" board output, and evidence the silence rule fired on your broken
connector test.

## 📚 Microsoft Learn

- [Monitor the health of your data connectors](https://learn.microsoft.com/en-us/azure/sentinel/monitor-data-connector-health)
- [Auditing and health monitoring in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/health-audit)
- [SentinelHealth table reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/sentinelhealth)

---

<div align="center">
<sub>

[⬅ Prev: 14 · API & codeless connectors](../14-api-and-codeless-connectors/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 16 · Retention, archive & data lake ➡](../16-retention-archive-and-data-lake/README.md)

</sub>
</div>
