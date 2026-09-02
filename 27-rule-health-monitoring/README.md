<div align="center">

# 🔍 Step 27 · Rule health monitoring

### *Catch the rule that silently stopped firing*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~25 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You can see every rule's last-run status, you've broken a rule on purpose and detected it via
`SentinelHealth`, and you have an alert on rule failures.

## 🧠 Why this step

A scheduled rule can fail every run — a renamed column, a Basic-tier table it can no longer query, a
query timeout, a deleted watchlist — and nothing tells you. The incident queue just goes quiet,
which looks like "a calm week".

## ✅ Prerequisites

- [Step 15](../15-ingestion-health-and-validation/README.md) — `SentinelHealth` enabled
- Several active rules

## 🧭 Where rule health lives

| Signal | Place |
|---|---|
| Per-rule last run | **Analytics → rule → Health** tab (or the **Health** column) |
| All rules, all runs | `SentinelHealth` where `SentinelResourceType == "Analytics Rule"` |
| Failure reasons | `SentinelHealth` `Description` / `ExtendedProperties` |
| Auto-disabled rules | Sentinel disables a rule after repeated failures — shows as **Auto disabled** |

## 🖱️ Do it — break one, then find it

1. **Analytics → DET-IDENTITY-001 → Edit → Set rule logic.** Change `TargetUserName` to
   `TargetUserNameXYZ` (a column that doesn't exist). Save.
2. Wait for two run cycles.
3. **Analytics → Active rules** — the **Health** column now shows **Failure** for that rule.
4. Open the rule → **Health** tab → read the error ("column 'TargetUserNameXYZ' not found").
5. Fix the column back. Confirm the next run is **Success**.

## 💻 Do it — query and alert

```kusto
// every rule failure in the last 7 days
SentinelHealth
| where TimeGenerated > ago(7d)
| where SentinelResourceType == "Analytics Rule"
| where Status != "Success"
| project TimeGenerated, RuleName = SentinelResourceName, Status,
          Reason = tostring(ExtendedProperties.Reason), Description
| sort by TimeGenerated desc
```

```kusto
// rules that have not produced a successful run recently
SentinelHealth
| where TimeGenerated > ago(2d) and SentinelResourceType == "Analytics Rule"
| summarize LastSuccess = maxif(TimeGenerated, Status == "Success"),
            LastRun = max(TimeGenerated) by SentinelResourceName
| where isnull(LastSuccess) or LastSuccess < ago(6h)
```

**Build the alert:** a scheduled rule (every 1h, lookback 2h) on the first query, threshold > 0,
severity Medium, title `OPS · Analytics rule failing`. Optionally route it to a different queue /
automation than security incidents.

## 🧪 Validate

Re-break the rule, and confirm:

- The **Health** column flips to Failure within ~2 cycles.
- The `SentinelHealth` query returns the failure with a readable reason.
- The `OPS · Analytics rule failing` rule raises an incident.

Then fix it and confirm the ops incident can be closed and doesn't reopen.

**You should see** a full loop: break → detected by health rule → fixed → healthy. That loop is the
deliverable.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Relying on "no incidents = all good" | Silence can mean every rule is failing |
| Not enabling `SentinelHealth` | No failure signal at all |
| Rule failures in the same queue as security incidents | Ops noise buries real alerts — separate them |
| Ignoring "Auto disabled" | Sentinel turned your detection off and you didn't notice |

## 🗒️ Log your run

`LOG.md` — the failure reason text, and evidence the health rule fired.

## 📚 Microsoft Learn

- [Monitor the health of your analytics rules](https://learn.microsoft.com/en-us/azure/sentinel/monitor-analytics-rule-integrity)
- [Auditing and health monitoring in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/health-audit)

---

<div align="center">
<sub>

[⬅ Prev: 26 · Tuning a noisy rule](../26-tuning-a-noisy-rule/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 28 · Analytics rules as code ➡](../28-analytics-rules-as-code/README.md)

</sub>
</div>
