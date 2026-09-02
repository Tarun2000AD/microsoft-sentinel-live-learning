<div align="center">

# 🛰️ Step 58 · Threat intelligence

### *Ingest indicators via TAXII / the upload API, and use them in detections*

[![Phase](https://img.shields.io/badge/Phase-Operate at scale-8957E5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~45 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-TI ingestion is free-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Indicators are flowing into `ThreatIntelligenceIndicator` (from a TAXII feed and/or the upload API),
a **TI matching** analytics rule is enabled, and you've fired it with a planted IOC.

## 🧠 Why this step

TI turns "was this IP ever seen in our logs?" into an automatic detection. Sentinel matches
incoming logs against your indicator set continuously; you also use indicators directly in custom
rules and hunts.

## ✅ Prerequisites

- [Step 07](../07-connectors-and-content-hub/README.md) — Threat Intelligence solution installed
- Some log volume to match against (`CommonSecurityLog`, `DnsEvents`, `SigninLogs`, `DeviceNetworkEvents`)
- A free TAXII feed (e.g. an OpenCTI/anomali test feed, or MISP) — or just use the upload API

## 🧭 The routes in

| Route | Best for | Table |
|---|---|---|
| **Threat Intelligence – TAXII** connector | Standing feeds (STIX 2.x over TAXII 2.x) | `ThreatIntelligenceIndicator` (+ newer `ThreatIntelIndicators`) |
| **Threat Intelligence upload API** | Push from your own TIP / scripts (STIX objects) | same |
| **Microsoft Defender Threat Intelligence** connector | Microsoft's own curated feed (licence-dependent) | same |
| **Manual** (portal → Threat intelligence → Add new) | One-off IOC during an incident | same |

## 🖱️ Do it — TAXII connector

1. **Data connectors → Threat Intelligence – TAXII → Open connector page.**
2. Friendly name, **API root URL**, **Collection ID**, username/password (if any), poll frequency,
   confidence threshold. Add.
3. Wait a poll cycle → `ThreatIntelligenceIndicator` populates.

## 💻 Do it — upload API (STIX)

```bash
TOKEN=$(az account get-access-token --resource https://management.azure.com --query accessToken -o tsv)
WS_SUB=<sub>; RG=rg-sentinel-lab; WS=law-sentinel-lab
curl -X POST "https://sentinelus.azure-api.net/<workspace-id>/threatintelligence-stix-objects/upload?api-version=2024-02-01-preview" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{
    "sourcesystem": "lab-upload",
    "stixobjects": [{
      "type": "indicator", "spec_version": "2.1", "id": "indicator--$(uuidgen)",
      "created": "2026-09-02T00:00:00.000Z", "modified": "2026-09-02T00:00:00.000Z",
      "name": "lab-test-bad-ip", "pattern": "[ipv4-addr:value = \x27198.51.100.23\x27]",
      "pattern_type": "stix", "valid_from": "2026-09-02T00:00:00Z",
      "labels": ["malicious-activity"], "confidence": 90
    }]
  }'
```

## 🖱️ Enable TI matching

**Analytics → Rule templates → "Microsoft Sentinel TI map ..." / "Threat Intelligence" rules** (one
per log type: IP, domain, URL, file hash). Enable **TI map IP entity to CommonSecurityLog** (or to
`DeviceNetworkEvents` / `DnsEvents` depending on your data). It runs on a schedule and matches
`ThreatIntelligenceIndicator` against those tables.

## 🧪 Validate

```kusto
ThreatIntelligenceIndicator
| where TimeGenerated > ago(7d) and Active == true
| summarize count() by SourceSystem, ThreatType = tostring(parse_json(Tags))
```

Plant the IOC: make your lab host connect to `198.51.100.23` (or query a "bad" domain), matching the
indicator you uploaded. On the next rule run:

```kusto
SecurityIncident
| where TimeGenerated > ago(2h) and Title has "TI map"
| project TimeGenerated, Title, Severity, IncidentNumber
```

```kusto
// use TI directly in a hunt
let badIps = ThreatIntelligenceIndicator
    | where Active == true and isnotempty(NetworkIP)
    | project NetworkIP;
CommonSecurityLog
| where TimeGenerated > ago(1d)
| where DestinationIP in (badIps) or SourceIP in (badIps)
```

**You should see** your indicator in `ThreatIntelligenceIndicator`, an incident when your lab
traffic matched it, and the direct-hunt query returning the same hit.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Enabling TI-map rules with no indicators | Rule runs, matches nothing, looks "fine" |
| Ingesting a huge low-confidence feed | Noise + false positives — set a confidence threshold |
| No indicator expiry | Stale IOCs keep firing on now-benign infrastructure |
| Only using the built-in TI-map rules | Also join TI in your own high-value rules and hunts |

## 🗒️ Log your run

`LOG.md` — the feed/upload used, indicator count, and the TI-match incident.

## 📚 Microsoft Learn

- [Threat intelligence in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/understand-threat-intelligence)
- [Connect threat intelligence feeds (TAXII, upload API, MDTI)](https://learn.microsoft.com/en-us/azure/sentinel/connect-threat-intelligence-taxii)
- [Use threat indicators in analytics rules](https://learn.microsoft.com/en-us/azure/sentinel/use-threat-indicators-in-analytics-rules)

---

<div align="center">
<sub>

[⬅ Prev: 57 · SOC optimization & coverage](../57-soc-optimization-and-coverage/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 59 · Anomaly & ML rules ➡](../59-anomaly-and-ml-rules/README.md)

</sub>
</div>
