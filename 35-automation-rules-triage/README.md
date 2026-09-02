<div align="center">

# 🔄 Step 35 · Automation rules for triage

### *Assign, tag, re-severity and suppress — with no code*

[![Phase](https://img.shields.io/badge/Phase-Automation-3FB950?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

A small set of ordered automation rules that: assign incidents to a queue, tag by data source,
downgrade a known-benign pattern to Closed/Benign, and add a standard task — all before an analyst
looks.

## 🧠 Why this step

Automation rules are the cheapest triage win. They run instantly, cost nothing, and remove the
repetitive first 60 seconds of every incident.

## ✅ Prerequisites

- [Step 29](../29-automation-rules-vs-playbooks/README.md)
- Some incident volume (run your sims, or enable a couple more templates)

## 🧭 The conditions & actions you have

**Conditions** (AND/OR groups): Title, Description, Severity, Status, Tactics, Analytics rule name,
Alert product name, Tag, Incident provider, and **entity properties** (e.g. IP address, Account
name, Host name, custom details).

**Actions**: Assign owner · Change status (+ classification/reason) · Change severity · Add tags ·
Add task · Run playbook(s).

**Order** matters: rules run ascending; a rule can be set so no lower-priority rules run after it.

## 🖱️ Do it — build four rules

| Order | Name | Trigger | Condition | Actions |
|---|---|---|---|---|
| 10 | `AR · Tag by source` | Incident created | Analytics rule name *contains* `SigninLogs` → tag `source:identity`; another for `SecurityEvent` → `source:endpoint` | Add tags |
| 20 | `AR · Suppress known scanner` | Incident created | Entity **IP address** *equals* `<your lab scanner IP>` **AND** Title *contains* `brute force` | Change status → Closed, classification **BenignPositive / "SecurityTestingTool"**; tag `auto-closed` |
| 30 | `AR · Assign High to Tier-2` | Incident created | Severity *equals* High | Assign owner → your Tier-2 group; add task "Confirm containment within 30 min" |
| 40 | `AR · Default triage task` | Incident created | Status *equals* New | Add task "Validate the alert against raw logs; document verdict" |

Build each: **Automation → Automation rules → Create → Automation rule**, set trigger, add
conditions, add actions, set Order, Save.

## 💻 Do it — automation rule as ARM (for step 55 later)

```json
{
  "type": "Microsoft.SecurityInsights/automationRules",
  "name": "<guid>",
  "properties": {
    "displayName": "AR - Suppress known scanner",
    "order": 20,
    "triggeringLogic": {
      "isEnabled": true, "triggersOn": "Incidents", "triggersWhen": "Created",
      "conditions": [
        { "conditionType": "Property", "conditionProperties": { "propertyName": "IncidentTitle", "operator": "Contains", "propertyValues": ["brute force"] } },
        { "conditionType": "Property", "conditionProperties": { "propertyName": "IncidentRelatedEntitiesIPAddress", "operator": "Equals", "propertyValues": ["<scanner-ip>"] } }
      ]
    },
    "actions": [
      { "order": 1, "actionType": "ModifyProperties", "actionConfiguration": { "status": "Closed", "classification": "BenignPositive", "classificationReason": "SecurityTestingTool" } },
      { "order": 2, "actionType": "AddIncidentTask", "actionConfiguration": { "title": "Auto-closed: scanner IP" } }
    ]
  }
}
```

## 🧪 Validate

Run: (a) the step-19 attack from your scanner IP, (b) the same from a different IP, (c) a High
incident.

```kusto
SecurityIncident
| where TimeGenerated > ago(1h)
| project IncidentNumber, Title, Severity, Status, Classification, Owner=tostring(Owner.assignedTo), Labels
```

**You should see**: (a) auto-closed with classification BenignPositive and `auto-closed` tag,
(b) open, tagged `source:endpoint`, with the default triage task, (c) assigned to Tier-2 with the
containment task. Check the **Order** worked — the suppress rule closed (a) before rule 40 added a
task, if you set "stop processing".

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| A broad "close benign" rule at Order 1 | Silently closes real incidents that happen to match |
| Suppressing by closing vs true suppression | Closing is auditable and reversible; prefer it, tagged |
| No expiration on temporary rules | They outlive the reason they existed |
| Assign-owner to an individual | Use a group/queue; people go on leave |

## 🗒️ Log your run

`LOG.md` — the three test incidents and how each was auto-handled. Export the rules to `artifacts/`.

## 📚 Microsoft Learn

- [Automate incident handling with automation rules](https://learn.microsoft.com/en-us/azure/sentinel/automate-incident-handling-with-automation-rules)
- [Create and use automation rules](https://learn.microsoft.com/en-us/azure/sentinel/create-manage-use-automation-rules)
- [Incident tasks in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/incident-tasks)

---

<div align="center">
<sub>

[⬅ Prev: 34 · Response actions with approval](../34-response-actions-with-approval/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 36 · Alert vs incident vs entity triggers ➡](../36-alert-vs-incident-vs-entity-triggers/README.md)

</sub>
</div>
