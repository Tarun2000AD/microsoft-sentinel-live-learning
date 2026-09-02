<div align="center">

# 🏹 Step 41 · The Hunting blade

### *Queries, MITRE, run-all, results — and how it differs from Analytics*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've run built-in hunting queries, saved a custom one, used **Run all queries**, and turned one of
your step-40 hypotheses into a saved hunting query.

## 🧠 Why this step

The Hunting blade is where proactive queries live. Unlike analytics rules, hunting queries **don't
create alerts** — they're for a human to run, review, and decide. Knowing the blade makes hunting
repeatable.

## ✅ Prerequisites

- [Step 40](../40-hunting-mindset-and-hypotheses/README.md) — three hypotheses written
- Content hub solutions installed (they bring hunting queries too)

## 🧭 Hunting vs Analytics

| | Hunting query | Analytics rule |
|---|---|---|
| Output | A result set you look at | An alert / incident |
| Cadence | Manual (or Livestream, step 43) | Scheduled / NRT |
| Tuning bar | Low — noisy is OK, you're exploring | High — must be precise |
| MITRE | Tagged, shown on the blade | Tagged, shown on the matrix |
| Promotion | → bookmark → incident, or → new analytics rule (step 49) | already a rule |

## 🖱️ Do it — portal

1. **Microsoft Sentinel → Hunting.** The **Queries** tab lists built-ins by data source and MITRE
   tactic. Columns show **Results** count and **Results delta** (change since last run).
2. Filter to tables you have (`SigninLogs`, `SecurityEvent`, `DeviceProcessEvents`). Select ~10
   relevant queries → **Run selected queries** (or **Run all queries**). Wait.
3. Sort by **Results** desc. Open the top few → **Run query** → **View results** in Logs.
4. **+ New query** → paste your `HUNT-ENDPOINT-001` KQL, name it `HUNT-ENDPOINT-001 · LOLBin network
   load`, set tactic **Defense Evasion**, technique **T1218**, description = your hypothesis. Save.
5. Note the **Livestream** and **Bookmark** buttons on the results — steps 42–43.

## 💻 Do it — save a hunting query as code

```json
{
  "type": "Microsoft.OperationalInsights/workspaces/savedSearches",
  "name": "law-sentinel-lab/HUNT-ENDPOINT-001",
  "properties": {
    "category": "Hunting Queries",
    "displayName": "HUNT-ENDPOINT-001 LOLBin network load",
    "query": "DeviceProcessEvents | where FileName in~ ('regsvr32.exe','rundll32.exe','mshta.exe') | where ProcessCommandLine has_any ('http://','https://','\\\\\\\\') | project TimeGenerated, DeviceName, AccountName, FileName, ProcessCommandLine, InitiatingProcessFileName",
    "tags": [
      { "name": "description", "value": "If a LOLBin loads code from a URL/UNC path it may be attacker execution." },
      { "name": "tactics", "value": "DefenseEvasion" },
      { "name": "techniques", "value": "T1218" }
    ]
  }
}
```

## 🧪 Validate

```kusto
// the blade reads saved hunting queries from here
_GetWatchlist  // no — hunting queries:
search in (SavedSearches) *   // not queryable; instead confirm in the portal Hunting → Queries list
```

In the portal: your query appears in **Hunting → Queries** with your MITRE tags, a **Run** button,
and a **Results** count after running. Run all built-in queries and record which three returned the
most results as hunt candidates.

**You should see** your custom hunting query listed and runnable, and a shortlist of built-in
queries worth investigating in steps 44–48.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Expecting hunting queries to alert | They don't — that's step 49 (promote to rule) |
| "Run all" on a huge workspace repeatedly | Slow and can be costly on Basic-tier tables |
| High Results count = incident | It's a *starting point*; triage before you escalate |
| Not tagging MITRE on custom queries | They don't show on the coverage matrix |

## 🗒️ Log your run

`LOG.md` — your saved query, and the top-3 built-in queries by results (with counts).

## 📚 Microsoft Learn

- [Hunt for threats with Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/hunting)
- [Create custom hunting queries](https://learn.microsoft.com/en-us/azure/sentinel/hunting#create-custom-hunting-queries)
- [Use hunts to run an end-to-end hunt](https://learn.microsoft.com/en-us/azure/sentinel/hunts)

---

<div align="center">
<sub>

[⬅ Prev: 40 · Hunting mindset & hypotheses](../40-hunting-mindset-and-hypotheses/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 42 · Bookmarks ➡](../42-bookmarks/README.md)

</sub>
</div>
