<div align="center">

# 🏹 Step 46 · Hunt: lateral movement

### *RDP/SMB spread, remote service creation, and new host pairs*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~45 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Run three lateral-movement hunts, simulate movement between two lab VMs, and bookmark the hits.

## 🧠 Why this step

Lateral movement is where an intrusion becomes a breach. The signal is rarely one bad event — it's
an account or host doing something it *has never done before*: logging into a new machine, creating
a remote service, mounting an admin share.

## ✅ Prerequisites

- Two Windows VMs (`vm-win-lab`, `vm-win-lab2`) with `SecurityEvent` or `DeviceLogonEvents`
- Both **isolated** — a private VNet, no public inbound

## 🧭 The three hunts

### 1️⃣ Account authenticating to a host it has never touched — T1021

```kusto
let lookback = 30d;
let recent = 1d;
let history =
    SecurityEvent
    | where TimeGenerated between (ago(lookback) .. ago(recent))
    | where EventID == 4624 and LogonType in (3, 10)
    | summarize by TargetUserName, Computer;
SecurityEvent
| where TimeGenerated > ago(recent)
| where EventID == 4624 and LogonType in (3, 10)
| summarize Logons = count(), FirstSeen = min(TimeGenerated) by TargetUserName, Computer, IpAddress = tostring(IpAddress)
| join kind=leftanti history on TargetUserName, Computer
| sort by FirstSeen asc
```

### 2️⃣ Remote service creation (PsExec-style) — T1021.002 / T1543.003

```kusto
SecurityEvent
| where TimeGenerated > ago(7d)
| where EventID == 7045      // A service was installed
| project TimeGenerated, Computer, ServiceName = tostring(parse_json(EventData).ServiceName),
          ImagePath = tostring(parse_json(EventData).ImagePath),
          AccountName = SubjectUserName
| where ImagePath has_any (@"\ADMIN$", "cmd.exe /c", "powershell", "%COMSPEC%", ".exe -")
     or ServiceName matches regex @"^[A-Za-z0-9]{8,}$"     // random-looking name
| sort by TimeGenerated desc
```

### 3️⃣ Admin share access spike from one source — T1021.002

```kusto
SecurityEvent
| where TimeGenerated > ago(7d)
| where EventID == 5140 and ShareName in~ (@"\\*\ADMIN$", @"\\*\C$")
| summarize Accesses = count(), Targets = dcount(Computer), TargetList = make_set(Computer, 10)
    by SubjectUserName, IpAddress = tostring(IpAddress), bin(TimeGenerated, 1h)
| where Targets >= 3
| sort by Targets desc
```

## 🖱️ Do it — simulate

From `vm-win-lab`, move to `vm-win-lab2` (both yours, in the lab VNet):

```powershell
# new-host logon
$cred = Get-Credential                       # a lab admin
New-PSSession -ComputerName vm-win-lab2 -Credential $cred

# admin share access
net use \\vm-win-lab2\C$ /user:LAB\adminuser
dir \\vm-win-lab2\C$\Windows\Temp

# remote service (then remove it)
sc.exe \\vm-win-lab2 create LabMoveTest binPath= "cmd.exe /c calc.exe"
sc.exe \\vm-win-lab2 delete LabMoveTest
```

## 🧪 Validate

Re-run the hunts after ingestion:

**You should see** the `adminuser`→`vm-win-lab2` pair in hunt 1 (leftanti = never seen before),
`LabMoveTest` in hunt 2, and the `C$` accesses in hunt 3. Baseline (before simulation): hunt 1
returns only genuinely new pairs from normal ops, hunts 2–3 return nothing.

Bookmark the chain as a **group** and promote to one incident — this is a multi-stage story, and
step 62's capstone reuses exactly this.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| `join kind=inner` instead of `leftanti` for "never seen" | Returns the opposite of what you want |
| History window too short | Everything looks "new" |
| Ignoring LogonType | Type 3 (network) and 10 (RemoteInteractive/RDP) matter; type 5 (service) is noise here |
| Not correlating the three hunts | Individually weak; together it's a clear movement chain |

## 🗒️ Log your run

`LOG.md` + `HUNT-LATERAL-00X.md`. Note the grouped incident number for the capstone.

## 📚 Microsoft Learn

- [Hunt for lateral movement](https://learn.microsoft.com/en-us/azure/sentinel/hunting)
- [SecurityEvent EventID reference (4624, 5140, 7045)](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [KQL join operator](https://learn.microsoft.com/en-us/kusto/query/join-operator)

---

<div align="center">
<sub>

[⬅ Prev: 45 · Hunt: endpoint](../45-hunt-endpoint/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 47 · Hunt: exfiltration ➡](../47-hunt-exfiltration/README.md)

</sub>
</div>
