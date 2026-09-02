<div align="center">

# 🏹 Step 48 · Hunt: cloud control plane

### *Role grants, Key Vault access, NSG opening, and diagnostic tampering*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~45 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Run four Azure control-plane hunts against `AzureActivity` / `AuditLogs` / `AzureDiagnostics`,
simulate each in your lab subscription, and bookmark.

## 🧠 Why this step

Once an attacker has cloud credentials, the control plane *is* the target: grant themselves Owner,
pull secrets from Key Vault, open an NSG to the internet, and turn off the logging that would catch
them. This hunt covers the moves.

## ✅ Prerequisites

- [Step 08](../08-azure-activity/README.md) — `AzureActivity`
- [Step 09](../09-microsoft-entra-id/README.md) — `AuditLogs`
- Key Vault diagnostic logs to `AzureDiagnostics` (enable on `kv-sentinel-lab`)

## 🧭 The four hunts

### 1️⃣ Privileged role assignment — T1098 / T1078.004

```kusto
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue =~ "Microsoft.Authorization/roleAssignments/write"
| where ActivityStatusValue == "Success"
| extend props = parse_json(tostring(Properties))
| extend RoleId = tostring(parse_json(tostring(props.requestbody)).properties.roleDefinitionId)
| where RoleId has_any ("8e3af657-a8ff-443c-a75c-2fe8c4bcb635",   // Owner
                        "b24988ac-6180-42a0-ab88-20f7382dd24c",   // Contributor
                        "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9")    // User Access Administrator
| project TimeGenerated, Caller, CallerIpAddress, ResourceGroup, _ResourceId, RoleId
```

### 2️⃣ Key Vault secret bulk read — T1552.001

```kusto
AzureDiagnostics
| where TimeGenerated > ago(7d)
| where ResourceProvider == "MICROSOFT.KEYVAULT"
| where OperationName in ("SecretGet","KeyGet","CertificateGet")
| summarize Reads = count(), Secrets = dcount(id_s), Names = make_set(id_s, 20)
    by identity_claim_upn_s, CallerIPAddress, bin(TimeGenerated, 1h)
| where Secrets >= 10 or Reads >= 50
| sort by Secrets desc
```

### 3️⃣ NSG rule opening a sensitive port to the internet — T1562.007 / T1021

```kusto
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue =~ "Microsoft.Network/networkSecurityGroups/securityRules/write"
| where ActivityStatusValue == "Success"
| extend body = parse_json(tostring(parse_json(tostring(Properties)).requestbody)).properties
| extend Access = tostring(body.access), Dir = tostring(body.direction),
         Src = tostring(body.sourceAddressPrefix), Ports = tostring(body.destinationPortRange)
| where Access =~ "Allow" and Dir =~ "Inbound"
| where Src in ("*","0.0.0.0/0","Internet")
| where Ports has_any ("22","3389","445","1433","3306","5985","5986") or Ports == "*"
| project TimeGenerated, Caller, CallerIpAddress, _ResourceId, Src, Ports
```

### 4️⃣ Diagnostic setting / logging disabled — T1562.008

```kusto
AzureActivity
| where TimeGenerated > ago(30d)
| where OperationNameValue has "microsoft.insights/diagnosticSettings/delete"
     or OperationNameValue has "microsoft.insights/diagnosticSettings/write"
| where ActivityStatusValue == "Success"
| project TimeGenerated, OperationNameValue, Caller, CallerIpAddress, _ResourceId
| sort by TimeGenerated desc
```

## 🖱️ Do it — simulate in the lab subscription

```bash
# 1. grant a test user Reader then Contributor on the RG (then remove)
az role assignment create --assignee hunt-test@YOURTENANT --role Contributor -g rg-sentinel-lab
az role assignment delete --assignee hunt-test@YOURTENANT --role Contributor -g rg-sentinel-lab

# 2. read several Key Vault secrets quickly
for s in $(az keyvault secret list --vault-name kv-sentinel-lab --query "[].name" -o tsv); do
  az keyvault secret show --vault-name kv-sentinel-lab --name "$s" >/dev/null; done

# 3. add then remove an NSG rule opening 3389 to the internet
az network nsg rule create -g rg-sentinel-lab --nsg-name <nsg> -n LabOpenRDP \
  --priority 400 --access Allow --direction Inbound --protocol Tcp \
  --source-address-prefixes Internet --destination-port-ranges 3389
az network nsg rule delete -g rg-sentinel-lab --nsg-name <nsg> -n LabOpenRDP

# 4. delete and re-create a diagnostic setting
az monitor diagnostic-settings delete --name entra-to-sentinel --resource "/providers/microsoft.aadiam/diagnosticSettings"
# (re-create it — see step 09)
```

## 🧪 Validate

Wait for `AzureActivity` (10–20 min). Re-run the hunts:

**You should see** each simulated action surface in its hunt, with your UPN as `Caller` and your IP
as `CallerIpAddress`. Baseline (before) returns only your legitimate lab-building activity — which
is itself a useful lesson in what "normal" for a small sub looks like. Bookmark the diagnostic-delete
and NSG-open ones; both are strong detections for step 49.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Parsing `Properties` inconsistently | `AzureActivity` nests JSON differently by operation — test your `parse_json` chain |
| Role GUIDs hard-coded and stale | Verify built-in role definition IDs against docs |
| Key Vault hunt without diagnostic logging enabled | `AzureDiagnostics` KV rows only exist if you turned them on |
| Treating your own admin activity as the baseline forever | In prod, a human admin doing this off-hours from a new IP *is* the hunt |

## 🗒️ Log your run

`LOG.md` + `HUNT-CLOUD-00X.md`. Re-enable any diagnostic setting you deleted — check step 15's board.

## 📚 Microsoft Learn

- [Hunt for threats across your Azure environment](https://learn.microsoft.com/en-us/azure/sentinel/hunting)
- [Azure built-in roles reference](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)
- [Monitor Key Vault with Azure Monitor](https://learn.microsoft.com/en-us/azure/key-vault/general/monitor-key-vault)

---

<div align="center">
<sub>

[⬅ Prev: 47 · Hunt: exfiltration](../47-hunt-exfiltration/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 49 · Hunt → detection ➡](../49-hunt-to-detection/README.md)

</sub>
</div>
