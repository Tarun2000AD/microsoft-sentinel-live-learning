<div align="center">

# 🛰️ Step 57 · SOC optimization & coverage

### *Use Sentinel's own recommendations to close gaps and cut waste*

[![Phase](https://img.shields.io/badge/Phase-Operate at scale-8957E5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've reviewed the **SOC optimization** recommendations, actioned one **coverage** and one
**data-value** recommendation, and built a small coverage scorecard.

## 🧠 Why this step

Sentinel now analyses your workspace and tells you: "you ingest table X but no rule uses it" (waste)
and "you have data for technique Y but no detection" (gap). It's a free quarterly review you should
run monthly.

## ✅ Prerequisites

- [Step 25](../25-mitre-attack-coverage/README.md), [56](../56-cost-engineering/README.md)
- Content hub solutions installed so there's something to recommend

## 🧭 What SOC optimization covers

| Type | Example recommendation | Action |
|---|---|---|
| **Coverage (threat-based)** | "You have sign-in data but no rule for password spray" | Enable a template / write a rule |
| **Data value** | "You ingest table X (Y GB/mo) that no rule, workbook or query touches" | Stop ingesting, or move to Basic, or add a detection |
| **Similar orgs** | "Orgs like yours also ingest / detect Z" | Consider the connector/rule |
| **Recommendations for MITRE gaps** | Specific tactic/technique with no coverage but available data | Deploy matching content |

## 🖱️ Do it — portal

1. **Microsoft Sentinel → SOC optimization** (also surfaced in the Defender portal).
2. Sort by **Value** / potential impact. Read each recommendation and its rationale.
3. **Action one coverage recommendation**: pick a threat scenario it flags (e.g. "AADSTS password
   spray"), follow its link to enable the relevant analytics rule template(s) or Content hub
   solution.
4. **Action one data-value recommendation**: for a table it says is unused, either add a detection
   that uses it, move it to Basic (step 16), or stop the connector. Document which and why.
5. Mark recommendations as **Completed** / dismiss with a reason.

## 💻 Do it — build your own coverage scorecard

```kusto
// detection coverage by MITRE tactic (from enabled rules)
let tactics = dynamic(["InitialAccess","Execution","Persistence","PrivilegeEscalation","DefenseEvasion",
  "CredentialAccess","Discovery","LateralMovement","Collection","CommandAndControl","Exfiltration","Impact"]);
SecurityAlert
| where TimeGenerated > ago(90d)
| mv-expand t = todynamic(Tactics) to typeof(string)
| summarize Rules = dcount(AlertName), Alerts90d = count() by Tactic = t
| join kind=rightouter (print Tactic = tactics | mv-expand Tactic to typeof(string)) on Tactic
| project Tactic, Rules = coalesce(Rules, 0), Alerts90d = coalesce(Alerts90d, 0)
| extend Status = case(Rules == 0, "❌ none", Rules == 1, "⚠️ thin", "✅ covered")
| sort by Rules asc
```

```kusto
// data you pay for but don't detect on
Usage
| where TimeGenerated > ago(30d) and IsBillable
| summarize GB = sum(Quantity)/1000 by DataType
| join kind=leftanti (
    // crude: tables referenced by enabled rule queries -- maintain this list or parse rule bodies
    print DataType = dynamic(["SigninLogs","SecurityEvent","AzureActivity","AuditLogs","DnsEvents","CommonSecurityLog"])
    | mv-expand DataType to typeof(string)
  ) on DataType
| sort by GB desc
```

## 🧪 Validate

Produce `artifacts/coverage-scorecard.md` with the tactic table and the unused-data table. Then:

- One coverage recommendation is **actioned** (new/enabled rule) — verify it in Analytics.
- One data-value recommendation is **actioned** (rule added, tier changed, or connector stopped) —
  verify in `Usage` / Analytics.
- Your step-25 MITRE matrix shows one more tactic lit than before.

**You should see** the scorecard, two actioned recommendations with evidence, and a measurable
coverage improvement.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Dismissing recommendations without reading the rationale | You miss real gaps / real waste |
| Actioning "coverage" by enabling untuned templates in bulk | Step 26 exists for a reason |
| Ignoring data-value recs because "we might need it" | Move to Basic/Aux instead of paying Analytics rate |
| Running SOC optimization once | It's a recurring review — monthly |

## 🗒️ Log your run

`LOG.md` + `artifacts/coverage-scorecard.md` + the two actioned recommendations.

## 📚 Microsoft Learn

- [SOC optimization in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/soc-optimization/soc-optimization-access)
- [SOC optimization reference](https://learn.microsoft.com/en-us/azure/sentinel/soc-optimization/soc-optimization-reference)
- [Understand security coverage by MITRE ATT&CK](https://learn.microsoft.com/en-us/azure/sentinel/mitre-coverage)

---

<div align="center">
<sub>

[⬅ Prev: 56 · Cost engineering](../56-cost-engineering/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 58 · Threat intelligence ➡](../58-threat-intelligence/README.md)

</sub>
</div>
