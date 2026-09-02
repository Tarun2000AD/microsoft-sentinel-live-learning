<div align="center">

# 🔍 Step 28 · Analytics rules as code

### *Export, version and redeploy a rule — no more portal-only detections*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~35 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Your custom rules are exported as ARM templates in this repo's `artifacts/`, you can redeploy them
to a clean state, and you understand the two supported paths (ARM/Bicep and the Sentinel
Repositories feature — full CI/CD is step 55).

## 🧠 Why this step

A detection that exists only as portal clicks can't be reviewed, rolled back, replicated to another
workspace, or recovered if someone deletes it. Rules are `Microsoft.SecurityInsights/alertRules`
ARM resources — treat them like code.

## ✅ Prerequisites

- [Step 19](../19-write-a-scheduled-rule/README.md)+ — you have custom rules
- `git` and the Azure CLI

## 🧭 Concepts in 60 seconds

| Path | Good for | Notes |
|---|---|---|
| **Export from portal** → ARM JSON | Snapshotting what exists | Analytics → Export (single rule or all) |
| **Author ARM/Bicep** by hand | New rules, PR review | `kind: Scheduled` / `NRT`, `properties` mirror the wizard |
| **Sentinel Repositories** | Team CI/CD across all content types | GitHub/Azure DevOps connection; step 55 |
| **`az sentinel alert-rule`** | Scripting, quick redeploy | Good for loops; ARM is more complete |

## 🖱️ Do it — export

1. **Analytics → Active rules** → select your custom rules → **Export** → save the JSON.
2. **Analytics → Export** (top bar) also exports *all* rules at once.
3. Drop the files in this step's `artifacts/rules/`.

## 💻 Do it — a clean Bicep for one rule

`artifacts/rules/det-identity-001.bicep`:

```bicep
param workspaceName string
param ruleGuid string = guid('DET-IDENTITY-001')

resource ws 'Microsoft.OperationalInsights/workspaces@2023-09-01' existing = { name: workspaceName }

resource rule 'Microsoft.SecurityInsights/alertRules@2024-03-01' = {
  scope: ws
  name: ruleGuid
  kind: 'Scheduled'
  properties: {
    displayName: 'DET-IDENTITY-001 Brute force then success'
    enabled: true
    severity: 'Medium'
    query: loadTextContent('../brute-force-then-success.kql')
    queryFrequency: 'PT1H'
    queryPeriod: 'PT1H10M'
    triggerOperator: 'GreaterThan'
    triggerThreshold: 0
    suppressionEnabled: true
    suppressionDuration: 'PT1H'
    tactics: [ 'CredentialAccess' ]
    techniques: [ 'T1110' ]
    entityMappings: [
      { entityType: 'Account', fieldMappings: [ { identifier: 'Name', columnName: 'TargetUserName' } ] }
      { entityType: 'IP', fieldMappings: [ { identifier: 'Address', columnName: 'IpAddress' } ] }
      { entityType: 'Host', fieldMappings: [ { identifier: 'HostName', columnName: 'Computer' } ] }
    ]
    incidentConfiguration: {
      createIncident: true
      groupingConfiguration: { enabled: true, lookbackDuration: 'PT5H', matchingMethod: 'Selected', groupByEntities: [ 'IP' ], reopenClosedIncident: false }
    }
  }
}
```

Deploy:

```bash
az deployment group create -g rg-sentinel-lab \
  --template-file artifacts/rules/det-identity-001.bicep \
  --parameters workspaceName=law-sentinel-lab
```

## 🧪 Validate

1. Note the rule's current settings. **Delete it in the portal.**
2. Run the deploy above.
3. Confirm the rule is back, identical:

```bash
az sentinel alert-rule show -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --rule-id "$(az sentinel alert-rule list -g rg-sentinel-lab --workspace-name law-sentinel-lab --query "[?contains(displayName,'DET-IDENTITY-001')].name" -o tsv)" \
  --query "{name:displayName, freq:queryFrequency, sev:severity, enabled:enabled}" -o table
```

**You should see** the rule recreated with the same frequency, severity, entity mappings and
grouping — deployment reproduced it exactly. Commit the Bicep/JSON to the repo.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Random `name` GUID each deploy | Creates a duplicate instead of updating — use a deterministic `guid()` |
| Query inline with escaped quotes | Ugly and error-prone — `loadTextContent()` a `.kql` file |
| Exported JSON has a stale `lastModifiedUtc` etc. | Strip read-only fields before redeploy |
| Editing in portal *and* code | Pick one source of truth (code) once step 55 is in place |

## 🗒️ Log your run

`LOG.md` — the delete-and-redeploy proof; commit `artifacts/rules/`.

## 📚 Microsoft Learn

- [Export and import analytics rules](https://learn.microsoft.com/en-us/azure/sentinel/import-export-analytics-rules)
- [Microsoft.SecurityInsights/alertRules ARM reference](https://learn.microsoft.com/en-us/azure/templates/microsoft.securityinsights/alertrules)
- [Deploy custom content from your repository](https://learn.microsoft.com/en-us/azure/sentinel/ci-cd)

---

<div align="center">
<sub>

[⬅ Prev: 27 · Rule health monitoring](../27-rule-health-monitoring/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 29 · Automation rules vs playbooks ➡](../29-automation-rules-vs-playbooks/README.md)

</sub>
</div>
