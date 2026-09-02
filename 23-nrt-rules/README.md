<div align="center">

# 🔍 Step 23 · Near-real-time (NRT) rules

### *Sub-minute detection — and the constraints that come with it*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~25 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

One working NRT rule, and a clear understanding of what NRT can't do so you don't try to force it.

## 🧠 Why this step

Some detections need to fire in ~1 minute, not up to an hour: a global admin role grant, a Key Vault
mass secret dump, disabling of security tooling. NRT rules run about once a minute on the most
recent data. The price is a strict feature set.

## ✅ Prerequisites

- [Step 22](../22-scheduling-lookback-and-coverage-gaps/README.md)
- `AuditLogs` (step 09) for the example

## 🧭 What NRT can and cannot do

| ✅ Can | ❌ Cannot |
|---|---|
| Run ~every 60s on the last minute of data | Look back further than ~a few minutes |
| Query a **single table** | `join` across tables (limited `union` only) |
| Use entity mapping, custom details, alert details | Use aggregations that need a wide time window |
| Trigger playbooks / automation rules | Use `bin()` over long windows, cross-workspace |
| ~50 NRT rules per workspace (check current limit) | Reference watchlists in some cases (verify) |

Rule of thumb: **NRT = "this single event, right now, is bad."** Anything needing correlation or
counting-over-time stays a scheduled rule.

## 🖱️ Do it — portal

1. **Analytics → Create → NRT query rule.**
2. Name `NRT · Global Admin role assigned`. Severity High. Tactic Privilege Escalation, T1098.
3. Query:

```kusto
AuditLogs
| where OperationName == "Add member to role"
| where TargetResources has "Global Administrator"
     or parse_json(tostring(TargetResources[0].modifiedProperties)) has "Global Administrator"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend Target = tostring(TargetResources[0].userPrincipalName)
| project TimeGenerated, Actor, Target, OperationName, Result
| extend AccountCustomEntity = Target, InitiatorEntity = Actor
```

4. Entity mapping: Account → `Target`. Custom details: `Actor`, `Result`.
5. Review + Create.

## 💻 Do it — CLI

```bash
az sentinel alert-rule create -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --rule-id $(uuidgen) \
  --nrt-alert-rule enabled=true display-name="NRT Global Admin role assigned" \
    severity=High trigger-operator=GreaterThan trigger-threshold=0 \
    query="$(cat artifacts/nrt-global-admin.kql)" tactics="PrivilegeEscalation"
```

## 🧪 Validate

Assign the **Global Administrator** role to a throwaway test user (`nrt-test@<tenant>`), then remove
it. Within ~2–3 minutes:

```kusto
SecurityAlert
| where TimeGenerated > ago(15m) and AlertName has "Global Admin role assigned"
| project TimeGenerated, AlertName, AlertSeverity, Entities
```

Compare the alert's `TimeGenerated` to the `AuditLogs` event time — the gap should be **1–3
minutes**, versus up to an hour for a scheduled rule.

**You should see** the incident within a few minutes, entity `Target` populated, `Actor` in custom
details. Then try adding a `join` to the query in the editor — the wizard rejects it, proving the
constraint.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Porting a scheduled correlation rule to NRT | The `join` / long window won't validate |
| Using NRT for noisy events | ~1 alert/min potential — reserve NRT for genuinely rare, genuinely urgent events |
| Expecting a lookback | NRT sees ~the last minute; a delayed log may be missed |
| Hitting the NRT rule count limit | Budget your NRT slots for the top-severity detections only |

## 🗒️ Log your run

`LOG.md` — the detection latency you measured (alert time − audit event time) and the rejected-join
screenshot.

## 📚 Microsoft Learn

- [Detect threats quickly with near-real-time (NRT) analytics rules](https://learn.microsoft.com/en-us/azure/sentinel/near-real-time-rules)
- [Create NRT detection rules](https://learn.microsoft.com/en-us/azure/sentinel/create-nrt-rules)

---

<div align="center">
<sub>

[⬅ Prev: 22 · Scheduling, lookback & coverage gaps](../22-scheduling-lookback-and-coverage-gaps/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 24 · Watchlists ➡](../24-watchlists/README.md)

</sub>
</div>
