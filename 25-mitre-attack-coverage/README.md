<div align="center">

# 🔍 Step 25 · MITRE ATT&CK coverage

### *See which techniques your rules touch — and which they don't*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~25 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've read the Sentinel MITRE ATT&CK blade, tagged your custom rules with the right
tactic/technique, and identified three coverage gaps to close.

## 🧠 Why this step

"How good is our detection coverage?" is a real question from real managers. The ATT&CK matrix turns
a pile of rules into a picture: which tactics are well covered, which have one flaky rule, which are
blank. It also drives what you build next.

## ✅ Prerequisites

- [Step 18](../18-enable-a-rule-from-template/README.md) and [19](../19-write-a-scheduled-rule/README.md) — several rules exist

## 🧭 Concepts in 60 seconds

- **Tactics** = the *why* (Initial Access, Persistence, Exfiltration…). **Techniques** = the *how*
  (T1078 Valid Accounts, T1110 Brute Force…). **Sub-techniques** = T1110.001 etc.
- The Sentinel **MITRE ATT&CK** blade colours each technique by how many **active analytics rules**
  and **hunting queries** reference it.
- Rules carry tactics/techniques in their metadata — only if you set them.
- Coverage ≠ effectiveness: a technique lit up by one untuned rule is not "covered".

## 🖱️ Do it — portal

1. **Microsoft Sentinel → MITRE ATT&CK.** Toggle the legend: **Active analytics rules**,
   **Hunting queries**, **Anomaly rules**, **NRT rules**.
2. Note the darkest cells (best covered) and the blank tactics.
3. For each custom rule you built (DET-IDENTITY-001, the NRT admin rule, the silence rule):
   **Analytics → rule → Edit → General → Tactics & techniques** → set them accurately.
   - DET-IDENTITY-001 → Credential Access / **T1110**
   - NRT Global Admin → Privilege Escalation / **T1098** (Account Manipulation)
   - "Source went quiet" → this is really *Defense Evasion / T1562.008* (disable cloud logs) if
     you framed it as detecting logging tamper; otherwise leave it untagged (it's ops health).
4. Refresh the matrix — your techniques are now lit.

## 💻 Do it — query your coverage

```kusto
// techniques covered by ENABLED rules, from rule metadata
SecurityAlert
| where TimeGenerated > ago(30d)
| mv-expand Tactic = todynamic(Tactics)
| summarize Rules = dcount(AlertName), Alerts = count() by tostring(Tactic)
| sort by Rules desc
```

```bash
az sentinel alert-rule list -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --query "[?enabled].{name:displayName, tactics:tactics, techniques:techniques}" -o json
```

## 🧪 Validate

**You should see** in the portal matrix at least Credential Access and Privilege Escalation now
showing "1+ active rule", and the KQL above returning your tactics with rule counts. Pick **three
blank tactics** relevant to your data (e.g. *Persistence*, *Exfiltration*, *Lateral Movement*) and
write them into `artifacts/coverage-gaps.md` as your build backlog — steps 44–48 (hunts) and 49
(hunt→detection) will fill them.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Leaving tactics unset on custom rules | The matrix shows a gap you don't actually have |
| Treating one rule per technique as "done" | Depth (multiple data sources, tuned) matters |
| Chasing 100% matrix coverage | Some techniques aren't observable with your data — be honest |
| Ignoring hunting-query coverage | Hunts count toward visibility even without alerts |

## 🗒️ Log your run

`LOG.md` + `artifacts/coverage-gaps.md` (your three prioritised gaps).

## 📚 Microsoft Learn

- [Understand security coverage by the MITRE ATT&CK framework](https://learn.microsoft.com/en-us/azure/sentinel/mitre-coverage)
- [MITRE ATT&CK — Enterprise matrix](https://attack.mitre.org/matrices/enterprise/)

---

<div align="center">
<sub>

[⬅ Prev: 24 · Watchlists](../24-watchlists/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 26 · Tuning a noisy rule ➡](../26-tuning-a-noisy-rule/README.md)

</sub>
</div>
