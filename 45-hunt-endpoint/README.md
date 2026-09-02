<div align="center">

# 🏹 Step 45 · Hunt: endpoint

### *LOLBins, encoded PowerShell, and persistence footholds*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~50 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Run three endpoint hunts against `DeviceProcessEvents` / `SecurityEvent`, simulate each safely on
`vm-win-lab`, and bookmark the hits.

## 🧠 Why this step

Attackers avoid dropping obvious malware. They live off the land — signed Windows binaries, base64
PowerShell, registry Run keys. These hunts find that.

## ✅ Prerequisites

- [Step 10](../10-defender-xdr/README.md) (`DeviceProcessEvents`) **or** [11](../11-windows-vm-ama-dcr/README.md) with process-creation auditing (4688) + command line logging
- `vm-win-lab` you can run commands on — **isolated, no public exposure**

## 🧭 The three hunts

### 1️⃣ LOLBin loading remote content — T1218

```kusto
DeviceProcessEvents
| where TimeGenerated > ago(7d)
| where FileName in~ ("regsvr32.exe","rundll32.exe","mshta.exe","installutil.exe","msiexec.exe","certutil.exe")
| where ProcessCommandLine has_any ("http://","https://","\\\\","ftp://")
     or (FileName =~ "certutil.exe" and ProcessCommandLine has_any ("-urlcache","-decode"))
| project TimeGenerated, DeviceName, AccountName, FileName, ProcessCommandLine,
          InitiatingProcessFileName, InitiatingProcessCommandLine
| sort by TimeGenerated desc
```

### 2️⃣ Encoded / obfuscated PowerShell — T1059.001

```kusto
DeviceProcessEvents
| where TimeGenerated > ago(7d)
| where FileName in~ ("powershell.exe","pwsh.exe")
| where ProcessCommandLine has_any ("-enc","-EncodedCommand","-e ","FromBase64String","-nop","-w hidden","IEX","Invoke-Expression","DownloadString","-ExecutionPolicy Bypass")
| extend b64 = extract(@"(?i)-e(?:nc(?:odedcommand)?)?\s+([A-Za-z0-9+/=]{20,})", 1, ProcessCommandLine)
| extend Decoded = iff(isnotempty(b64), base64_decode_tostring(b64), "")
| project TimeGenerated, DeviceName, AccountName, ProcessCommandLine, Decoded, InitiatingProcessFileName
```

*(`base64_decode_tostring` of a UTF-16LE payload shows every other byte as a dot — still readable
enough to triage.)*

### 3️⃣ Persistence via Run keys / scheduled tasks / services — T1547.001, T1053.005, T1543.003

```kusto
union
 (DeviceRegistryEvents
  | where TimeGenerated > ago(7d)
  | where RegistryKey has_any (@"\CurrentVersion\Run", @"\CurrentVersion\RunOnce", @"\Winlogon\Shell", @"\Winlogon\Userinit")
  | project TimeGenerated, DeviceName, Type="RegRunKey", Detail=strcat(RegistryValueName, " = ", RegistryValueData), InitiatingProcessFileName),
 (DeviceProcessEvents
  | where TimeGenerated > ago(7d)
  | where FileName in~ ("schtasks.exe","sc.exe") and ProcessCommandLine has_any ("create","/create","config")
  | project TimeGenerated, DeviceName, Type="TaskOrService", Detail=ProcessCommandLine, InitiatingProcessFileName)
| sort by TimeGenerated desc
```

## 🖱️ Do it — simulate safely on the lab VM

```powershell
# 1. LOLBin (benign target)
certutil.exe -urlcache -f https://example.com/ %TEMP%\test.txt

# 2. encoded PowerShell (prints a string, does nothing else)
$p = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes('Write-Output SENTINEL-LAB-TEST'))
powershell.exe -nop -w hidden -enc $p

# 3. persistence (then REMOVE it)
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v LabTest /d "calc.exe" /f
schtasks /create /tn LabTestTask /tr "calc.exe" /sc onlogon /f
# cleanup:
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v LabTest /f
schtasks /delete /tn LabTestTask /f
```

## 🧪 Validate

Wait ~10–15 min for `DeviceProcessEvents` / `DeviceRegistryEvents`, re-run the hunts:

**You should see** each simulated action in its hunt's output — `certutil -urlcache`, the decoded
`SENTINEL-LAB-TEST` string, and the `LabTest` Run key + `LabTestTask`. Bookmark each with Host +
Account entities. Confirm your **baseline** (before simulation) had none of these.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| No command-line logging | 4688 without `Include command line` shows the binary, not the args |
| `has` vs `has_any` vs `contains` | `has` is token-based and faster; `contains` is substring |
| Hunting only `powershell.exe` | Attackers rename it or use `pwsh`, `.NET`, WMI — widen carefully |
| Forgetting to clean up persistence | You left a foothold in your lab |

## 🗒️ Log your run

`LOG.md` + `HUNT-ENDPOINT-00X.md` with baselines, the simulated commands, and hits. Confirm cleanup.

## 📚 Microsoft Learn

- [Advanced hunting: DeviceProcessEvents](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-deviceprocessevents-table)
- [LOLBAS project](https://lolbas-project.github.io/)
- [KQL string operators](https://learn.microsoft.com/en-us/kusto/query/datatypes-string-operators)

---

<div align="center">
<sub>

[⬅ Prev: 44 · Hunt: identity](../44-hunt-identity/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 46 · Hunt: lateral movement ➡](../46-hunt-lateral-movement/README.md)

</sub>
</div>
