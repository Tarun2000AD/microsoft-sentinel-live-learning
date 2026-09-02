<div align="center">

# 🏹 Step 49 · Hunt → detection

### *Turn a hunt that found something into a scheduled rule*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~40 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Take one of your hunt queries from steps 44–48, harden it, and ship it as a tuned analytics rule
with entity mapping and MITRE tags — closing one of the coverage gaps from step 25.

## 🧠 Why this step

A hunt that finds a real technique but stays a manual query is a gap that reopens the moment you
stop looking. Promotion is how hunting *improves* detection coverage over time.

## ✅ Prerequisites

- [Steps 44–48](../44-hunt-identity/README.md) — at least one hunt with a real hit
- [Step 19](../19-write-a-scheduled-rule/README.md), [26](../26-tuning-a-noisy-rule/README.md)

## 🧭 Hunt query ≠ detection query

| Hunt query | Detection query |
|---|---|
| Wide time window (7–30d) | Window = run frequency + buffer |
| Noisy is fine | Must be precise — tune first |
| No entities required | Entity mapping mandatory |
| You read every row | It creates incidents unattended |
| No dedupe needed | Dedupe / one row per entity per window |
| No `timestamp` column | Needs `timestamp` for the incident timeline |

## 🖱️ Do it — promote the DNS-tunneling hunt (step 47)

1. **Hunting → your `HUNT-EXFIL-001` query → ⋯ → Create analytics rule** (this pre-fills the wizard
   with the query + MITRE tags).
2. **Harden the query:**

```kusto
let lookback = 1h;
DnsEvents
| where TimeGenerated > ago(lookback)
| where QueryType in ("A","AAAA","TXT","NULL","CNAME")
| extend Sub = tostring(split(Name, ".")[0])
| extend Domain = strcat(tostring(split(Name, ".")[-2]), ".", tostring(split(Name, ".")[-1]))
| where Domain !in (dynamic(["microsoft.com","windows.com","azure.com","office.com","msftncsi.com"]))  // allowlist
| summarize DistinctSubdomains = dcount(Name), AvgSubLen = avg(strlen(Sub)),
            TxtQ = countif(QueryType == "TXT"), Queries = count()
    by ClientIP, Domain, bin(TimeGenerated, lookback)
| where DistinctSubdomains > 80 and AvgSubLen > 25
| extend timestamp = TimeGenerated, IPCustomEntity = ClientIP
```

3. **Scheduling:** every 1h, lookback 1h 10m, suppression 3h.
4. **Entity mapping:** IP → `ClientIP`. Custom details: `Domain`, `DistinctSubdomains`, `AvgSubLen`.
5. **MITRE:** Exfiltration / Command and Control · T1071.004, T1048.
6. **Incident grouping:** by IP + Domain, 6h.
7. Severity Medium (High if `TxtQ` > 0 — set via Alert details).

## 🧪 Validate

1. **Baseline** — run the hardened query in Logs over the last day. Confirm **0 rows** (or only
   known-good, which you then allowlist).
2. **Fire it** — re-run the step-47 DNS simulation (300 long random subdomains under
   `lab-exfil-test.example`).
3. Wait for the rule run:

```kusto
SecurityIncident
| where TimeGenerated > ago(2h) and Title has "DNS tunneling"
| project TimeGenerated, Title, Severity, Status, IncidentNumber
```

4. **Regression** — confirm your earlier detections (DET-IDENTITY-001 etc.) still fire and the new
   rule doesn't add noise over 24h.

**You should see** a clean baseline, one incident on the simulation with the IP entity and custom
details populated, and step 25's MITRE matrix now showing Exfiltration covered.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Shipping the hunt query unchanged | 30-day window in an hourly rule = huge cost + dupes |
| No allowlist | CDNs and telemetry domains trip it daily |
| No baseline check before enabling | You find out it's noisy via 200 incidents |
| Forgetting to update the `HUNT-*.md` outcome | The hunt's lifecycle is undocumented |

## 🗒️ Log your run

`LOG.md` + a `DET-EXFIL-001.md` + update `HUNT-EXFIL-001.md` **Outcome** to
"Turned into detection DET-EXFIL-001". Commit the hardened `.kql`.

## 📚 Microsoft Learn

- [Create an analytics rule from a hunting query](https://learn.microsoft.com/en-us/azure/sentinel/hunting#create-an-analytics-rule-from-a-hunting-query)
- [Custom analytics rules to detect threats](https://learn.microsoft.com/en-us/azure/sentinel/detect-threats-custom)

---

<div align="center">
<sub>

[⬅ Prev: 48 · Hunt: cloud control plane](../48-hunt-cloud-control-plane/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 50 · Notebooks & MSTICPy ➡](../50-notebooks-and-msticpy/README.md)

</sub>
</div>
