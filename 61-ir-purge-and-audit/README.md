<div align="center">

# 🛰️ Step 61 · IR, purge & auditing Sentinel itself

### *Data purge, GDPR requests, and watching the watchers*

[![Phase](https://img.shields.io/badge/Phase-Operate at scale-8957E5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~35 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've run a targeted data **purge**, built an audit view of **who changed Sentinel** (rules,
connectors, incidents), and written an incident-response runbook for "the SOC platform itself is
compromised".

## 🧠 Why this step

The SIEM holds sensitive data and is a high-value target. You need to: honour data-subject deletion
requests, prove what an analyst did during an incident, and have a plan for when your detection
platform is the thing under attack.

## ✅ Prerequisites

- [Step 15](../15-ingestion-health-and-validation/README.md) — `SentinelHealth` / auditing enabled
- Owner on the workspace (purge is a privileged operation)

## 🧭 Three capabilities

### 1️⃣ Data purge (GDPR "right to erasure")

- `POST .../workspaces/<ws>/purge` with a predicate on a table (e.g. `AzureActivity` where
  `Caller == "user@x"`).
- Asynchronous; returns a purge ID you poll.
- Purge is **rate-limited** and **audited**. It removes matching rows from analytics tables.
- Archived data: purge covers it too, but restored/search-job result tables are separate.

### 2️⃣ Auditing Sentinel changes

| What changed | Where to see it |
|---|---|
| Analytics rule created/edited/deleted | `SentinelAudit` / `LAQueryLogs` + `AzureActivity` on `Microsoft.SecurityInsights/alertRules` |
| Connector enabled/disabled | `AzureActivity` on `Microsoft.SecurityInsights/dataConnectors`, `SentinelHealth` |
| Incident touched (status, owner, comments) | `SecurityIncident` history columns + `SentinelAudit` |
| Who ran a query | `LAQueryLogs` (enable it) |
| Workspace/RBAC change | `AzureActivity`, `AuditLogs` |

### 3️⃣ IR runbook: "the SOC platform is compromised"

Scenarios: an analyst account is phished; a playbook MI is abused to disable users; someone deletes
rules/connectors to go dark; the workspace is targeted for purge abuse.

## 🖱️ Do it

**Purge:**

```bash
TOKEN=$(az account get-access-token --resource https://management.azure.com --query accessToken -o tsv)
WS=$(az monitor log-analytics workspace show -g rg-sentinel-lab -n law-sentinel-lab --query id -o tsv)

curl -X POST "https://management.azure.com$WS/purge?api-version=2023-09-01" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{ "table": "CheckoutApp_CL",
        "filters": [ { "column": "User", "operator": "==", "value": "svc-batch" } ] }'
# -> returns { "operationId": "..." }; poll:
curl "https://management.azure.com$WS/operations/<operationId>?api-version=2023-09-01" -H "Authorization: Bearer $TOKEN"
```

**Audit view (KQL):**

```kusto
// every Sentinel config change in 30 days, who and from where
AzureActivity
| where TimeGenerated > ago(30d)
| where OperationNameValue has "Microsoft.SecurityInsights" or OperationNameValue has "Microsoft.OperationalInsights"
| where OperationNameValue has_any ("write","delete")
| project TimeGenerated, Caller, CallerIpAddress, OperationNameValue, ActivityStatusValue, _ResourceId
| sort by TimeGenerated desc
```

```kusto
// analytics rules disabled or deleted -- a "going dark" indicator
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue matches regex @"alertRules/(write|delete)"
| project TimeGenerated, Caller, CallerIpAddress, OperationNameValue
```

## 🧪 Validate

1. **Purge**: ingest a few `CheckoutApp_CL` rows for `svc-batch` (step 13), purge them, poll to
   `Completed`, confirm:

```kusto
CheckoutApp_CL | where User == "svc-batch" | count   // -> 0 after purge completes
```

2. **Audit**: make a change (disable a rule), then confirm it appears in the `AzureActivity` audit
   query with your `Caller` and IP.
3. **Runbook**: write `artifacts/ir-runbook-platform-compromise.md` covering, for each scenario:
   detection signal, containment (revoke sessions, disable MI, freeze rule changes), eradication,
   and recovery (redeploy content from Git — step 55 is why you can).

**You should see** a completed purge with a 0-row confirmation, an audit trail that names you as the
actor, and a runbook that leans on the as-code work from steps 28/38/55 for recovery.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Purge with a loose predicate | Deletes more than the request — purges are irreversible |
| No audit on query access (`LAQueryLogs` off) | Can't answer "who looked at the exec's mailbox data" |
| Runbook assumes the SIEM is trustworthy during its own incident | Have an out-of-band comms + a second pair of eyes on rule changes |
| No as-code backup of content | "Someone deleted all our rules" becomes a rebuild, not a redeploy |

## 🗒️ Log your run

`LOG.md` — the purge operation result, the audit query output (redacted), and
`artifacts/ir-runbook-platform-compromise.md`.

## 📚 Microsoft Learn

- [Manage personal data — data purge in a Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/personal-data-mgmt)
- [Auditing and health monitoring in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/health-audit)
- [Audit Microsoft Sentinel queries and activities](https://learn.microsoft.com/en-us/azure/sentinel/audit-sentinel-data)

---

<div align="center">
<sub>

[⬅ Prev: 60 · SIEM migration](../60-siem-migration/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 62 · Capstone ➡](../62-capstone/README.md)

</sub>
</div>
