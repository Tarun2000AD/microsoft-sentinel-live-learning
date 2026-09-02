<div align="center">

# 📥 Step 09 · Connect Microsoft Entra ID

### *Sign-in and audit logs — the identity control plane*

[![Phase](https://img.shields.io/badge/Phase-Data onboarding-1F6FEB?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~15 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-per GB (SigninLogs can be chatty)-E3B341?style=flat-square)](#)

</div>

---

## 🎯 Goal

`SigninLogs` and `AuditLogs` (and ideally the non-interactive / service-principal sign-in tables)
are flowing, and you can see your own logins.

## 🧠 Why this step

Most cloud intrusions are identity intrusions. `SigninLogs` gives you MFA results, conditional
access outcomes, risk levels, device and location per authentication. `AuditLogs` gives you
directory changes — role grants, app consents, group membership. Together they answer a huge share
of real investigations, which is why so many detection templates depend on them.

## ✅ Prerequisites

- [Step 07](../07-connectors-and-content-hub/README.md) — Microsoft Entra ID solution installed
- **Global Administrator** or **Security Administrator** in the tenant (required to set the diagnostic setting)
- An Entra ID **P1/P2** licence for the risk-related tables (`AADUserRiskEvents`, `RiskyUsers`) — optional for the lab

## 🧭 The tables

| Table | Contents | Licence |
|---|---|---|
| `SigninLogs` | Interactive user sign-ins | Free |
| `AADNonInteractiveUserSignInLogs` | Token refresh, client apps | Free (can be *very* high volume) |
| `AADServicePrincipalSignInLogs` | App / workload identity sign-ins | Free |
| `AADManagedIdentitySignInLogs` | Managed identity sign-ins | Free |
| `AuditLogs` | Directory changes | Free |
| `AADUserRiskEvents`, `RiskyUsers` | Identity Protection risk | P2 |
| `AADProvisioningLogs` | SCIM provisioning | Free |

## 🖱️ Do it — portal

1. **Microsoft Sentinel → Data connectors → Microsoft Entra ID → Open connector page.**
2. It links to **Entra ID → Diagnostic settings**. Click **Add diagnostic setting**.
3. Name `entra-to-sentinel`. Select log categories: **SignInLogs, AuditLogs,
   NonInteractiveUserSignInLogs, ServicePrincipalSignInLogs, ManagedIdentitySignInLogs,
   ProvisioningLogs**. For the lab, skip `NonInteractive` if you want to keep volume down.
4. Destination: **Send to Log Analytics workspace** → `law-sentinel-lab` → **Save**.

## 💻 Do it — CLI

```bash
WS=$(az monitor log-analytics workspace show -g rg-sentinel-lab -n law-sentinel-lab --query id -o tsv)

az monitor diagnostic-settings create \
  --name entra-to-sentinel \
  --resource "/providers/microsoft.aadiam/diagnosticSettings" \
  --workspace "$WS" \
  --logs '[
    {"category":"SignInLogs","enabled":true},
    {"category":"AuditLogs","enabled":true},
    {"category":"ServicePrincipalSignInLogs","enabled":true},
    {"category":"ManagedIdentitySignInLogs","enabled":true},
    {"category":"ProvisioningLogs","enabled":true}
  ]'
```

> The Entra diagnostic setting resource path is special (`microsoft.aadiam`). If the CLI form
> fights you, the portal path above is reliable.

## 🧪 Validate

Sign out and back in once to generate an event, wait ~15 minutes:

```kusto
SigninLogs
| where TimeGenerated > ago(1h)
| project TimeGenerated, UserPrincipalName, AppDisplayName, ResultType, ResultDescription,
          ConditionalAccessStatus, tostring(LocationDetails.city), tostring(DeviceDetail.operatingSystem)
| sort by TimeGenerated desc
```

```kusto
AuditLogs
| where TimeGenerated > ago(1d)
| project TimeGenerated, OperationName, tostring(InitiatedBy.user.userPrincipalName),
          tostring(TargetResources[0].displayName), Result
| sort by TimeGenerated desc
```

**You should see** your own sign-in with `ResultType == 0` (success), and in `AuditLogs` the user
creations and role assignments you did in step 05.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Enabling `NonInteractiveUserSignInLogs` in a big tenant | Can be 10–50x `SigninLogs` volume — cost |
| Expecting risk tables without P2 | `AADUserRiskEvents` stays empty on free/P1 |
| Only one diagnostic setting slot | Entra allows a limited number; consolidate categories into one |
| Forgetting this needs a tenant-level admin | Contributor on the workspace is not enough |

## 🗒️ Log your run

`LOG.md` — categories selected, whether you took `NonInteractive`, and the first `SigninLogs` result
(UPN/IP redacted).

## 📚 Microsoft Learn

- [Connect Microsoft Entra ID data to Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/connect-azure-active-directory)
- [SigninLogs schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/signinlogs)
- [AuditLogs schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/auditlogs)

---

<div align="center">
<sub>

[⬅ Prev: 08 · Azure Activity](../08-azure-activity/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 10 · Defender XDR ➡](../10-defender-xdr/README.md)

</sub>
</div>
