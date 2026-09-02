<div align="center">

# 🔍 Step 19 · Write a scheduled rule from scratch

### *Your first detection that is yours — brute force then success*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~45 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-negligible-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

A scheduled analytics rule you authored — KQL, thresholds, entity mapping, scheduling — fires an
incident on activity you generate, and stays quiet on your baseline.

## 🧠 Why this step

Templates run out. Every SOC engineer writes detections. This is the core skill: turn a hypothesis
("many failures then a success from the same IP = credential stuffing hit") into a rule that fires
correctly.

## ✅ Prerequisites

- [Step 18](../18-enable-a-rule-from-template/README.md) — you've seen the rule wizard
- `SecurityEvent` (from step 11) **or** `SigninLogs` (step 09)
- [Step 04](../04-kql-survival-kit/README.md) — KQL basics

## 🧭 Design the detection first

| Field | Decision |
|---|---|
| Hypothesis | ≥ 10 failed logons then ≥ 1 success, same source IP, same target account, within 1 hour |
| Data source | `SecurityEvent` (EventID 4625 fail, 4624 success) |
| Run frequency | every 1 hour |
| Lookback | 1 hour (matches frequency — no gap, no overlap) |
| Threshold | results > 0 |
| Entities | Account = `TargetUserName`, IP = `IpAddress`, Host = `Computer` |
| Severity | Medium |
| MITRE | Credential Access · T1110 (Brute Force) |

## 🖱️ Do it — the KQL

```kusto
let failures = SecurityEvent
    | where TimeGenerated > ago(1h) and EventID == 4625
    | summarize FailCount = count(), FirstFail = min(TimeGenerated)
        by TargetUserName, IpAddress = tostring(IpAddress), Computer;
let successes = SecurityEvent
    | where TimeGenerated > ago(1h) and EventID == 4624 and LogonType in (2, 3, 10)
    | summarize SuccessCount = count(), FirstSuccess = min(TimeGenerated)
        by TargetUserName, IpAddress = tostring(IpAddress), Computer;
failures
| join kind=inner successes on TargetUserName, IpAddress, Computer
| where FailCount >= 10 and FirstSuccess > FirstFail
| project FirstFail, FirstSuccess, TargetUserName, IpAddress, Computer, FailCount, SuccessCount
| extend timestamp = FirstSuccess, AccountCustomEntity = TargetUserName,
         IPCustomEntity = IpAddress, HostCustomEntity = Computer
```

In the wizard:
- **Analytics → Create → Scheduled query rule.**
- General: name `DET-IDENTITY-001 · Brute force followed by success`, severity Medium, tactic
  Credential Access, technique T1110.
- Set rule logic: paste the query. **Entity mapping**: Account → `TargetUserName`, IP →
  `IpAddress`, Host → `Computer`. Query scheduling: every `1h`, lookback `1h`.
- Incident settings: create incidents, group alerts on the same entities for 5 hours.
- Review + Create.

## 💻 Do it — CLI

```bash
az sentinel alert-rule create -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --rule-id $(uuidgen) \
  --scheduled-alert-rule enabled=true display-name="DET-IDENTITY-001 Brute force then success" \
    severity=Medium query-frequency=PT1H query-period=PT1H \
    trigger-operator=GreaterThan trigger-threshold=0 \
    query="$(cat artifacts/brute-force-then-success.kql)" \
    tactics="CredentialAccess"
```

Save the KQL to `artifacts/brute-force-then-success.kql` in this folder.

## 🧪 Validate

1. **Baseline check** — run the query in Logs now. Expect **0 rows** (no attack yet).
2. **Generate the attack** — from one source, fail RDP/SMB logon to `vm-win-lab` as `testvictim`
   12 times, then log on successfully once.
3. Wait for the next rule run (or **Analytics → rule → ⋯ → Run** if available), then:

```kusto
SecurityIncident
| where TimeGenerated > ago(2h) and Title has "DET-IDENTITY-001"
| project TimeGenerated, Title, Severity, Status, IncidentNumber
```

Open the incident → confirm **Entities** shows the account, IP and host.

**You should see**: baseline 0, then exactly one incident after the simulated attack, with all
three entities mapped. If it fires on your baseline, raise the threshold or tighten `LogonType`.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| No `timestamp` / entity columns in `project` | Entity mapping and incident timeline break |
| `join` default kind | Use `kind=inner` explicitly |
| Lookback ≠ frequency | Gap (lookback < freq) or duplicate alerts (lookback ≫ freq) |
| Threshold tuned to the lab | 10 failures is fine here; production may need 30+ and an allowlist |
| Testing only that it fires | Also prove it *doesn't* on normal logons |

## 🗒️ Log your run

`LOG.md` + `artifacts/brute-force-then-success.kql` + a `DET-IDENTITY-001.md` from
[`_templates/DETECTION-TEMPLATE.md`](../_templates/DETECTION-TEMPLATE.md).

## 📚 Microsoft Learn

- [Create custom analytics rules to detect threats](https://learn.microsoft.com/en-us/azure/sentinel/detect-threats-custom)
- [Map data fields to entities in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/map-data-fields-to-entities)
- [SecurityEvent schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/securityevent)

---

<div align="center">
<sub>

[⬅ Prev: 18 · Enable a rule from a template](../18-enable-a-rule-from-template/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 20 · Entity mapping & custom details ➡](../20-entity-mapping-and-custom-details/README.md)

</sub>
</div>
