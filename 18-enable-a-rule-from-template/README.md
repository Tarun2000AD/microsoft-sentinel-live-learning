<div align="center">

# 🔍 Step 18 · Enable a rule from a template

### *Ship your first working detection in five minutes*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~15 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-negligible (query runs)-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

At least two analytics rules from templates are **active**, tuned to your lookback/frequency, and
one has produced (or is ready to produce) an incident.

## 🧠 Why this step

Templates are maintained, MITRE-mapped, entity-mapped starting points. Enabling a few good ones
gives immediate coverage and shows you the rule wizard before you build one from scratch in step 19.

## ✅ Prerequisites

- [Step 09](../09-microsoft-entra-id/README.md) — `SigninLogs` flowing
- [Step 11](../11-windows-vm-ama-dcr/README.md) — `SecurityEvent` flowing (for the second rule)
- [Step 05](../05-rbac-and-roles/README.md) — Sentinel Contributor

## 🧭 Concepts in 60 seconds

- A template can be **used more than once** (create several rules from it with different scopes).
- The wizard has tabs: **General**, **Set rule logic** (query, entity mapping, query scheduling),
  **Incident settings** (grouping), **Automated response**, **Review**.
- Templates ship disabled; enabling is a decision per template.
- After enabling, the template shows **In use** and you get notified when Microsoft updates it.

## 🖱️ Do it — portal

1. **Analytics → Rule templates.** Search **"Sign-in from IP that fails to authenticate multiple
   times"** (or "Brute force attempt against a user account"). Open it → **Create rule**.
   - General: keep the name, severity Medium, tactics pre-filled.
   - Set rule logic: read the KQL. Query scheduling → **Run every 1 hour**, **lookback 1 hour** (or
     match the template's own comment).
   - Entity mapping is pre-filled — note the Account / IP mappings.
   - Review + Create.
2. Repeat with **"Excessive Windows logon failures"** / **"Multiple authentication failures
   followed by a success"** (needs `SecurityEvent`).
3. **Analytics → Active rules** — both now show **Enabled**, last-run pending.

## 💻 Do it — CLI

```bash
# list templates, grab one's name (GUID)
az sentinel alert-rule template list -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --query "[?contains(displayName,'Brute force')].{name:name, title:displayName}" -o table

# create a scheduled rule FROM a template
az sentinel alert-rule create -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --rule-id $(uuidgen) \
  --scheduled-alert-rule enabled=true display-name="Brute force attempt (from template)" \
    severity=Medium query-frequency=PT1H query-period=PT1H trigger-operator=GreaterThan trigger-threshold=0 \
    alert-rule-template-name="<template-guid>"
```

## 🧪 Validate

Generate the activity: fail sign-in for a test user 10+ times (portal sign-in page, wrong
password), or fail RDP logon to `vm-win-lab` repeatedly. Then wait for the rule's next run:

```kusto
SecurityAlert
| where TimeGenerated > ago(4h)
| where AlertName has "Brute force" or AlertName has "logon failures"
| project TimeGenerated, AlertName, AlertSeverity, Entities
```

```kusto
SecurityIncident
| where TimeGenerated > ago(4h)
| project TimeGenerated, Title, Severity, Status, AlertIds
```

Also **Analytics → the rule → Health** tab: last run **Success**, results count > 0.

**You should see** a `SecurityAlert` row and a matching `SecurityIncident`, with entities (the
account and source IP) populated. If the rule ran Success with 0 results, your test activity didn't
land in the lookback window — check `SecurityEvent`/`SigninLogs` directly and re-run inside the
window.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Enabling 50 templates at once | Alert flood before you've tuned anything — enable ~5, tune, expand |
| Lookback shorter than run frequency | Coverage gap — step 22 |
| Not generating test activity | "Success, 0 results" ≠ working; you haven't proven it fires |
| Ignoring the template's `requiredDataConnectors` | Rule enabled on a table with no data never fires |

## 🗒️ Log your run

`LOG.md` — the two rules enabled, the test activity you generated, and the incident it produced
(entities redacted).

## 📚 Microsoft Learn

- [Create scheduled analytics rules from templates](https://learn.microsoft.com/en-us/azure/sentinel/create-analytics-rule-from-template)
- [Manage analytics rule templates](https://learn.microsoft.com/en-us/azure/sentinel/manage-analytics-rule-templates)

---

<div align="center">
<sub>

[⬅ Prev: 17 · Analytics rule types](../17-analytics-rule-types/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 19 · Write a scheduled rule ➡](../19-write-a-scheduled-rule/README.md)

</sub>
</div>
