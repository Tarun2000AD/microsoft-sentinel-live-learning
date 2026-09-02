<div align="center">

# 🧱 Step 03 · Navigating Sentinel

### *Learn every blade and what it is actually for*

[![Phase](https://img.shields.io/badge/Phase-Foundations-8661C5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~20 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You can open any Sentinel blade and say in one sentence what it does, which later step covers it,
and which table it reads or writes.

## 🧠 Why this step

The Sentinel left-hand menu has ~20 items grouped into four sections. Every later step lives in one
of them. Five minutes of orientation now saves a lot of "where was that?" later.

## ✅ Prerequisites

- [Step 02](../02-enable-sentinel/README.md) — Sentinel enabled

## 🧭 The menu, section by section

### General

| Blade | What it's for | Step |
|---|---|---|
| **Overview** | At-a-glance: incidents, events, data, automation health | — |
| **Logs** | The KQL query editor over every workspace table | `04` |
| **Search** | Long-running search jobs across analytics + archived data | `16` |
| **News & guides** | Getting-started content and the data connector wizard entry | `07` |

### Threat management

| Blade | What it's for | Step |
|---|---|---|
| **Incidents** | The analyst work queue — correlated groups of alerts | `20`, `21` |
| **Workbooks** | Dashboards over Sentinel data (Azure Monitor workbook engine) | `57` |
| **Hunting** | Saved proactive queries, run manually or promoted to livestream | `41` |
| **Notebooks** | Jupyter notebooks (MSTICPy) against the workspace | `50` |
| **Entity behavior** | UEBA — per-user / per-host behavior and anomalies | `51` |
| **Threat intelligence** | Indicator management; TI feeds and the upload API | `58` |
| **MITRE ATT&CK** | Coverage matrix — which techniques your rules touch | `25` |

### Content management

| Blade | What it's for | Step |
|---|---|---|
| **Content hub** | Install solutions: bundled connectors + rules + workbooks + playbooks | `07` |
| **Repositories** | Connect a Git repo and deploy Sentinel content via CI/CD | `55` |
| **Community** | Link to the Sentinel GitHub repo |  — |

### Configuration

| Blade | What it's for | Step |
|---|---|---|
| **Data connectors** | Connect data sources; see connector health | `07`–`14` |
| **Analytics** | Create and manage detection rules (active + templates) | `17`–`27` |
| **Watchlists** | Reference lists used in rules and queries | `24` |
| **Automation** | Automation rules + the playbooks gallery | `29`–`39` |
| **Settings** | Workspace settings, UEBA toggle, entity settings, pricing, data lake | `06`, `51` |

## 🖱️ Do it — a 10-minute walk

1. **Overview** — note the four tiles are all zero. Screenshot for your baseline.
2. **Logs** — run `search * | take 10` (yes it errors politely with "no tables") and `Usage | take 5`.
3. **Analytics → Rule templates** — scroll. Hundreds of templates, none active. This is step `18`.
4. **Data connectors** — the gallery. Count how many say "Connected" (zero). This is step `07`.
5. **Automation** — empty. **Content hub** — search "Azure Activity", don't install yet.
6. **Settings → Settings tab** — find the **UEBA** section and the **Auditing and health monitoring**
   toggle. Leave both off for now (steps `51` and `27`).

## 🧪 Validate

In **Logs**, run:

```kusto
union withsource=TableName *
| summarize Rows=count() by TableName
| sort by Rows desc
```

**You should see** only a handful of tables (`Usage`, `Operation`, maybe `Heartbeat` with 0 rows) —
proof the workspace is essentially empty and every later ingestion step will visibly change this
list.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Living only in the Overview dashboard | It hides connector and rule *health*; those have their own blades |
| Confusing **Hunting** with **Analytics** | Hunting queries don't create alerts; analytics rules do (step `41`) |
| Confusing **Content hub** with **Data connectors** | Content hub *installs* the connector; Data connectors *configures* it |

## 🗒️ Log your run

`LOG.md` — attach the baseline Overview screenshot (redact the workspace name/subscription).

## 📚 Microsoft Learn

- [What is Microsoft Sentinel?](https://learn.microsoft.com/en-us/azure/sentinel/overview)
- [Microsoft Sentinel feature tour](https://learn.microsoft.com/en-us/azure/sentinel/whats-new)

---

<div align="center">
<sub>

[⬅ Prev: 02 · Enable Sentinel](../02-enable-sentinel/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 04 · KQL survival kit ➡](../04-kql-survival-kit/README.md)

</sub>
</div>
