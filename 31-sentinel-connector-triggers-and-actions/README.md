<div align="center">

# 🔄 Step 31 · The Microsoft Sentinel connector

### *Every trigger and action the playbook connector gives you*

[![Phase](https://img.shields.io/badge/Phase-Automation-3FB950?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-~cents-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've used the key Sentinel connector actions in a playbook: get incident, update incident, add
comment, add task, get entities, run a Log Analytics query, and alert/entity triggers — and you
know what each returns.

## 🧠 Why this step

Playbooks live or die on knowing this connector. Most real playbooks are: trigger → **entities** →
enrich → **update incident** / **add comment** / **add task**. This step is the reference you build
by using it.

## ✅ Prerequisites

- [Step 30](../30-first-playbook-notify/README.md)

## 🧭 The connector surface

### Triggers

| Trigger | Fires on | Gives you |
|---|---|---|
| **Microsoft Sentinel incident** | Incident created/updated (via automation rule) | Full incident object + entities + alerts |
| **Microsoft Sentinel alert** | Alert (attached to an analytics rule) | Single alert + its entities |
| **Microsoft Sentinel entity** | Manual run from an entity page | One entity |

### Actions (most-used)

| Action | Use |
|---|---|
| **Alert — Get incident** | Fetch latest incident state by ARM ID |
| **Update incident** | Set status, severity, owner, classification, tags |
| **Add comment to incident (V3)** | Post markdown back to the incident |
| **Add task to incident** | Add a checklist item for the analyst |
| **Entities — Get Accounts / IPs / Hosts / URLs / FileHashes** | Split the incident's entities by type into arrays |
| **Run query and list results / visualize** | Execute KQL against the workspace from the playbook |
| **Update watchlist / add watchlist item** | Push an IOC into a watchlist |// verify availability
| **Alerts — Get** | Enumerate alerts in the incident |

## 🖱️ Do it — build a "triage helper" playbook

`PB-Triage-Helper` (incident trigger):

1. **Entities — Get IPs** → input `@{triggerBody()?['object']?['properties']?['relatedEntities']}`
   (or the trigger's Entities). Output: array of IP entities.
2. **Entities — Get Accounts** → same input.
3. **Microsoft Sentinel — Run query and list results:**

```kusto
SigninLogs
| where TimeGenerated > ago(7d)
| where UserPrincipalName in ('@{items(...)}')   // loop or join the account list
| summarize SignIns = count(), Countries = make_set(Location), Apps = make_set(AppDisplayName)
    by UserPrincipalName
```

4. **Add comment to incident (V3):** a markdown table of the account's 7-day sign-in summary.
5. **Add task to incident:** "Confirm the source IP is not a known VPN egress."
6. **Update incident:** add tag `triage-helper-ran`.

## 🧪 Validate

Attach `PB-Triage-Helper` to an automation rule (or run it manually on an incident). Then:

```kusto
SecurityIncident
| where TimeGenerated > ago(1h) and Title has "DET-IDENTITY-001"
| project IncidentNumber, Labels, Comments, TaskCount = array_length(todynamic(tostring(Tasks)))
```

Open the incident: the comment has the sign-in summary table, a task appears in the **Tasks** pane,
and the `triage-helper-ran` tag is set.

**You should see** all three artifacts on the incident and a Succeeded run. Inspect the run's
**"Run query and list results"** output to see the raw shape the connector returns (a `value` array).

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Using "Get Accounts" output as a string | It's an array — loop it or `join` in KQL |
| Querying with a personal connection | Rate-limited and breaks on offboarding — managed identity, step 32 |
| Huge KQL results into a comment | Truncate / summarize; comments have size limits |
| Forgetting incidents update-trigger loops | An "update incident" action can re-trigger the rule — guard with tags/conditions |

## 🗒️ Log your run

`LOG.md` — the connector output shapes you observed; export `PB-Triage-Helper` to `artifacts/`.

## 📚 Microsoft Learn

- [Microsoft Sentinel connector reference (Logic Apps)](https://learn.microsoft.com/en-us/connectors/azuresentinel/)
- [Use triggers and actions in Microsoft Sentinel playbooks](https://learn.microsoft.com/en-us/azure/sentinel/playbook-triggers-actions)

---

<div align="center">
<sub>

[⬅ Prev: 30 · Your first playbook (notify)](../30-first-playbook-notify/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 32 · Playbook managed identity & permissions ➡](../32-playbook-managed-identity-and-permissions/README.md)

</sub>
</div>
