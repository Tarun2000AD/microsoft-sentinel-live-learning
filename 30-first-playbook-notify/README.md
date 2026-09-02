<div align="center">

# 🔄 Step 30 · Your first playbook — notify

### *Post an incident to Teams / email the moment it's created*

[![Phase](https://img.shields.io/badge/Phase-Automation-3FB950?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~40 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-~fractions of a cent per run-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

A playbook (Logic App) with the **Sentinel incident trigger** that sends a formatted notification,
wired to fire on your DET-IDENTITY-001 incidents via an automation rule.

## 🧠 Why this step

Notification is the "hello world" of playbooks and you'll reuse the skeleton for every later one:
incident trigger → parse fields → do something → comment back on the incident.

## ✅ Prerequisites

- [Step 05](../05-rbac-and-roles/README.md) — playbook permissions granted to Sentinel
- [Step 29](../29-automation-rules-vs-playbooks/README.md)
- A Teams channel you can post to, or an email address

## 🧭 Concepts in 60 seconds

- A playbook = Logic App (Consumption plan) + the **Microsoft Sentinel** connector as trigger.
- Trigger types: **incident**, **alert**, **entity** (step 36). Use **incident** here.
- The trigger output gives you incident properties, entities, and the incident ARM ID (needed to
  comment back).
- Connections (Teams, Office 365, Sentinel) authenticate as **you** by default — step 32 switches
  to managed identity.

## 🖱️ Do it — portal

1. **Automation → Create → Playbook with incident trigger.**
   - Name `PB-Notify-Incident`, resource group `rg-sentinel-lab`, region = your region.
   - Connections: it adds **Microsoft Sentinel**. Continue.
2. In the designer, **+ New step**:
   - **Microsoft Teams → Post message in a chat or channel** → Team + Channel → message:

```
🚨 Sentinel incident: @{triggerBody()?['object']?['properties']?['title']}
Severity: @{triggerBody()?['object']?['properties']?['severity']}
Number:  @{triggerBody()?['object']?['properties']?['incidentNumber']}
Link:    @{triggerBody()?['object']?['properties']?['incidentUrl']}
```

   *(or **Office 365 Outlook → Send an email (V2)** instead.)*
3. **+ New step → Microsoft Sentinel → Add comment to incident (V3)**:
   - Incident ARM ID: `@{triggerBody()?['object']?['id']}`
   - Comment: `Notified SOC channel at @{utcNow()}.`
4. **Save.**

## 💻 Do it — wire it up

- **Automation → Automation rules → Create:**
  - Trigger: *When incident is created*
  - Condition: *Analytics rule name* **contains** `DET-IDENTITY-001`
  - Action: *Run playbook* → `PB-Notify-Incident`
  - Order 10. Save.
- If prompted, **grant permission** for the automation rule to run playbooks in `rg-sentinel-lab`.

## 🧪 Validate

Re-run the step-19 brute-force simulation. Within a minute or two of the incident appearing:

- ✅ Teams/email message arrives with the real title, severity and number.
- ✅ The incident has a **comment** "Notified SOC channel…".
- ✅ **Logic App → Runs history** shows one **Succeeded** run.

```kusto
SecurityIncident
| where TimeGenerated > ago(1h) and Title has "DET-IDENTITY-001"
| project TimeGenerated, IncidentNumber, Comments
```

**You should see** the comment count ≥ 1 and a green run in history. A failed run: open it, read
which action failed (usually the Teams connection needs authorising).

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Playbook created but no automation rule / attachment | It never triggers |
| Wrong trigger (alert vs incident) | `triggerBody()` shape differs; expressions break |
| Skipping the "grant permissions" prompt | Run fails with authorization error |
| Hard-coding the incident ID | Use the trigger's dynamic `id` |
| Personal connection used forever | Breaks when you're offboarded — step 32 |

## 🗒️ Log your run

`LOG.md` — the run-history screenshot (redact the callback URL if visible) + the incident comment.
Export the playbook ARM to `artifacts/` (step 38 formalises this).

## 📚 Microsoft Learn

- [Tutorial: Use playbooks with automation rules](https://learn.microsoft.com/en-us/azure/sentinel/tutorial-respond-threats-playbook)
- [Microsoft Sentinel connector for Logic Apps](https://learn.microsoft.com/en-us/connectors/azuresentinel/)
- [Create and manage playbooks](https://learn.microsoft.com/en-us/azure/sentinel/automation/create-playbooks)

---

<div align="center">
<sub>

[⬅ Prev: 29 · Automation rules vs playbooks](../29-automation-rules-vs-playbooks/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 31 · The Sentinel connector ➡](../31-sentinel-connector-triggers-and-actions/README.md)

</sub>
</div>
