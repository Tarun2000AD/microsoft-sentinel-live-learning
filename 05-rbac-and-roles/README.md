<div align="center">

# 🧱 Step 05 · RBAC and roles

### *Assign the four Sentinel roles the way a real SOC would*

[![Phase](https://img.shields.io/badge/Phase-Foundations-8661C5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You have at least two test identities — one "analyst", one "engineer" — with the correct built-in
Sentinel roles, scoped to the resource group, and you have confirmed the analyst *cannot* do
engineer things.

## 🧠 Why this step

Sentinel RBAC is layered on Azure RBAC. If you only ever use your Owner account you will never
notice that a rule needs `Microsoft Sentinel Contributor`, a playbook attachment needs
`Microsoft Sentinel Playbook Operator` + Logic App permissions, and an analyst should never have
either. Getting this wrong in production is how analysts end up as subscription Owners.

## ✅ Prerequisites

- [Step 02](../02-enable-sentinel/README.md) — Sentinel enabled
- Ability to create test users in your tenant (or invite guests)

## 🧭 The four built-in roles

| Role | Can | Cannot | Give to |
|---|---|---|---|
| **Microsoft Sentinel Reader** | View data, incidents, workbooks, rules | Change anything, run playbooks | Auditors, managers |
| **Microsoft Sentinel Responder** | Reader + manage incidents (assign, status, severity, tasks) | Create/edit analytics rules | Tier-1 / Tier-2 analysts |
| **Microsoft Sentinel Contributor** | Responder + create/edit rules, workbooks, hunting queries | Manage workspace RBAC, billing | Detection engineers |
| **Microsoft Sentinel Playbook Operator** | Run playbooks manually, attach them to rules | Edit the Logic App itself | Analysts who need to trigger response |

Extra facts that trip people up:

- Running a playbook also needs the **Microsoft Sentinel Automation Contributor** role assigned to
  the *Sentinel service principal* on the playbook's resource group (Sentinel prompts you to grant
  this once).
- Reading raw logs can also be granted with the generic **Log Analytics Reader**; resource-context
  RBAC (`enableLogAccessUsingOnlyResourcePermissions`) lets VM owners see only their own logs.
- Assign at the **resource group** scope, not subscription, so the blast radius is the lab.

## 🖱️ Do it — portal

1. **Entra ID → Users → New user** → create `analyst1@<tenant>` and `engineer1@<tenant>`
   (no roles, no licences needed).
2. **rg-sentinel-lab → Access control (IAM) → Add role assignment**:
   - `Microsoft Sentinel Responder` → `analyst1`
   - `Microsoft Sentinel Playbook Operator` → `analyst1`
   - `Microsoft Sentinel Contributor` → `engineer1`
3. **Microsoft Sentinel → Settings → Settings → Playbook permissions** → **Configure permissions**
   → select `rg-sentinel-lab` → grant. (This assigns Automation Contributor to the Sentinel SP.)

## 💻 Do it — CLI

```bash
RG=$(az group show -n rg-sentinel-lab --query id -o tsv)
ANALYST=$(az ad user show --id analyst1@YOURTENANT --query id -o tsv)
ENGINEER=$(az ad user show --id engineer1@YOURTENANT --query id -o tsv)

az role assignment create --assignee $ANALYST  --role "Microsoft Sentinel Responder"          --scope $RG
az role assignment create --assignee $ANALYST  --role "Microsoft Sentinel Playbook Operator"  --scope $RG
az role assignment create --assignee $ENGINEER --role "Microsoft Sentinel Contributor"        --scope $RG
```

## 🧪 Validate

```bash
az role assignment list --scope $RG --query "[].{who:principalName, role:roleDefinitionName}" -o table
```

Then **sign in as `analyst1`** (private browser window) and confirm:

- ✅ Can open **Incidents**, change an incident's status and owner.
- ❌ **Analytics → Create** is greyed out / returns *authorization failed*.
- ✅ Can open a playbook's **Run** action (once playbooks exist in step 30).

**You should see** the analyst able to *work* incidents but unable to *author detections* — that
split is the whole point.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Giving analysts `Contributor` "to be safe" | They can now silently change or disable detections |
| Assigning roles at subscription scope | Blast radius is everything you own, not the lab |
| Forgetting the playbook-permissions grant | Playbooks attached to rules fail with a permissions error at run time |
| Using `Owner` for daily work | You never discover which specific role a task actually needs |

## 🗒️ Log your run

`LOG.md` — the `az role assignment list` output (UPNs redacted) and the analyst's blocked-action
screenshot.

## 📚 Microsoft Learn

- [Roles and permissions in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/roles)
- [Manage access to Microsoft Sentinel data by resource](https://learn.microsoft.com/en-us/azure/sentinel/resource-context-rbac)
- [Grant Microsoft Sentinel permission to run playbooks](https://learn.microsoft.com/en-us/azure/sentinel/tutorial-respond-threats-playbook#configure-permissions)

---

<div align="center">
<sub>

[⬅ Prev: 04 · KQL survival kit](../04-kql-survival-kit/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 06 · Cost model and budget ➡](../06-cost-model-and-budget/README.md)

</sub>
</div>
