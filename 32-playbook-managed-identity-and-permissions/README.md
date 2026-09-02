<div align="center">

# 🔄 Step 32 · Playbook managed identity & permissions

### *Stop authenticating playbooks as a person*

[![Phase](https://img.shields.io/badge/Phase-Automation-3FB950?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~35 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Your playbooks authenticate the Sentinel connection (and any Azure action) with the Logic App's
**system-assigned managed identity**, holding only the roles it needs.

## 🧠 Why this step

Default connections run as the user who created them. When that user leaves, or their token
expires, every playbook silently breaks. Managed identity has no password, no expiry, and gets
least-privilege RBAC you can audit.

## ✅ Prerequisites

- [Step 31](../31-sentinel-connector-triggers-and-actions/README.md) — a playbook doing Sentinel actions
- Owner/User Access Administrator on `rg-sentinel-lab` (to assign roles)

## 🧭 What each action needs

| Playbook does | Identity needs |
|---|---|
| Add comment / update incident / add task | **Microsoft Sentinel Responder** on the workspace RG |
| Run a Log Analytics query | **Log Analytics Reader** (or Sentinel Reader) on the workspace |
| Update / add watchlist items | **Microsoft Sentinel Contributor** |
| Post to Teams / send mail | The **Teams / Office 365** connector — still an OAuth connection (no MI support), use a service mailbox |
| Disable an Entra user (step 34) | A Graph permission on the MI: `User.EnableDisableAccount` / `User.ReadWrite.All` (app role) |
| Isolate a device via Defender | `Machine.Isolate` app role on the MI for WindowsDefenderATP / Graph Security |
| Add an NSG deny rule | **Network Contributor** scoped to the NSG |

## 🖱️ Do it — portal

1. **Logic App (`PB-Triage-Helper`) → Identity → System assigned → Status On → Save.** Copy the
   **Object (principal) ID**.
2. **rg-sentinel-lab → Access control (IAM) → Add role assignment:**
   - **Microsoft Sentinel Responder** → assign to → **Managed identity** → your Logic App.
   - **Log Analytics Reader** → same.
3. **Logic App designer → the Microsoft Sentinel connection → Change connection → Connect with
   managed identity → Create new** (name `sentinel-mi`). Re-point each Sentinel action to it.
4. Delete the old user-based connection resource.

## 💻 Do it — CLI

```bash
LA=$(az resource show -g rg-sentinel-lab -n PB-Triage-Helper --resource-type Microsoft.Logic/workflows --query identity.principalId -o tsv)
RG=$(az group show -n rg-sentinel-lab --query id -o tsv)
WS=$(az monitor log-analytics workspace show -g rg-sentinel-lab -n law-sentinel-lab --query id -o tsv)

az role assignment create --assignee-object-id $LA --assignee-principal-type ServicePrincipal \
  --role "Microsoft Sentinel Responder" --scope $RG
az role assignment create --assignee-object-id $LA --assignee-principal-type ServicePrincipal \
  --role "Log Analytics Reader" --scope $WS
```

For Graph app roles (used in step 34), assign an **app role** to the MI:

```bash
GRAPH_SP=$(az ad sp list --filter "appId eq '00000003-0000-0000-c000-000000000000'" --query "[0].id" -o tsv)
ROLE_ID=$(az ad sp show --id $GRAPH_SP --query "appRoles[?value=='User.ReadWrite.All'].id | [0]" -o tsv)
az rest --method POST \
  --uri "https://graph.microsoft.com/v1.0/servicePrincipals/$LA/appRoleAssignments" \
  --body "{\"principalId\":\"$LA\",\"resourceId\":\"$GRAPH_SP\",\"appRoleId\":\"$ROLE_ID\"}"
```

## 🧪 Validate

```bash
az role assignment list --assignee $LA --all -o table
```

Run the playbook manually on an incident. Confirm:

- ✅ Succeeded run, comment/task still posted — using `sentinel-mi`.
- ✅ **API connections** in the RG no longer include a personal `azuresentinel` connection.
- Remove one role (e.g. Responder) and re-run → the "add comment" action **fails with 403**,
  proving the MI (not your account) is now the identity. Re-add the role.

**You should see** the playbook fully functional with only the MI + its scoped roles, and a clean
403 when you under-permission it.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Assigning roles at subscription scope | Over-privileged MI; scope to the RG / workspace / NSG |
| Leaving the old user connection | It's still there as a fallback and still breaks confusingly |
| Expecting Teams/Outlook to use MI | Those connectors don't — use a dedicated service account |
| Granting `Owner` to "make it work" | Now a playbook bug can delete resources |

## 🗒️ Log your run

`LOG.md` — the `az role assignment list` for the MI (IDs redacted) and the 403 test.

## 📚 Microsoft Learn

- [Authenticate playbooks to Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/authenticate-playbooks-to-sentinel)
- [Managed identities for Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/create-managed-service-identity)
- [Assign Graph app roles to a managed identity](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/how-to-assign-app-role-managed-identity)

---

<div align="center">
<sub>

[⬅ Prev: 31 · The Sentinel connector](../31-sentinel-connector-triggers-and-actions/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 33 · Enrich an incident ➡](../33-enrich-an-incident/README.md)

</sub>
</div>
