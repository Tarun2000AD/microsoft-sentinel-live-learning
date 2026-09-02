<div align="center">

# 🧱 Step 04 · KQL survival kit

### *The 12 operators you will use every day, and nothing else yet*

[![Phase](https://img.shields.io/badge/Phase-Foundations-8661C5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~45 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You can read any Sentinel detection query, and write a filtering-and-summarizing query from a blank
editor without looking anything up.

## 🧠 Why this step

Every detection, every hunt, every workbook, every playbook condition in this repo is KQL. You do
not need the whole language — you need about a dozen operators fluently. This step drills those on
the `Usage` and `Operation` tables that already exist in your empty workspace.

## ✅ Prerequisites

- [Step 02](../02-enable-sentinel/README.md) — Sentinel enabled (so **Logs** works)

## 🧭 The 12

| Operator | Does | Example |
|---|---|---|
| `where` | filter rows | `where TimeGenerated > ago(1h)` |
| `project` / `project-away` | pick / drop columns | `project TimeGenerated, Operation` |
| `extend` | add a computed column | `extend Hour = bin(TimeGenerated, 1h)` |
| `summarize` | aggregate | `summarize count() by bin(TimeGenerated, 1h)` |
| `count` | quick row count | `Usage | count` |
| `sort` / `top` | order / order+limit | `top 10 by Quantity desc` |
| `take` / `limit` | sample rows (no order) | `take 20` |
| `distinct` | unique combinations | `distinct DataType` |
| `join` | combine two tables on a key | `join kind=inner (…) on Computer` |
| `union` | stack tables/results | `union SigninLogs, AuditLogs` |
| `parse` / `extract` | pull fields out of a string | `extract("user=([^ ]+)", 1, RawData)` |
| `let` | name a value or subquery | `let window = 24h;` |

Plus three time helpers you'll type constantly: `ago(1d)`, `bin(TimeGenerated, 1h)`, `now()`.

## 🖱️ Do it — run these in Logs, in order

```kusto
// 1. what is being ingested, and how much
Usage
| where TimeGenerated > ago(7d)
| summarize TotalGB = sum(Quantity) / 1000 by DataType
| sort by TotalGB desc
```

```kusto
// 2. control-plane operations on the workspace itself
Operation
| where TimeGenerated > ago(7d)
| summarize count() by OperationCategory, Detail
| sort by count_ desc
```

```kusto
// 3. shape of activity over time — the pattern behind every "spike" detection
Usage
| where TimeGenerated > ago(24h)
| summarize GB = sum(Quantity)/1000 by bin(TimeGenerated, 1h)
| render timechart
```

```kusto
// 4. let + join: is any data type suddenly quiet?
let recent = Usage | where TimeGenerated > ago(1d)  | summarize r = sum(Quantity) by DataType;
let prior  = Usage | where TimeGenerated between (ago(2d) .. ago(1d)) | summarize p = sum(Quantity) by DataType;
recent | join kind=leftouter prior on DataType
| extend DropPct = round(100.0 * (p - r) / p, 1)
| where DropPct > 50
```

```kusto
// 5. parse a string field (you will do this constantly with syslog/CEF in step 12)
print RawData = "src=10.0.0.5 dst=10.0.0.9 action=deny proto=tcp spt=44321 dpt=445"
| extend SrcIp = extract(@"src=([0-9\.]+)", 1, RawData),
         DstPort = toint(extract(@"dpt=([0-9]+)", 1, RawData)),
         Action = extract(@"action=(\w+)", 1, RawData)
```

## 🧪 Validate

Write this **from scratch**, no copy-paste: *"For the last 3 days, show the top 5 data types by GB
ingested, as a bar chart."*

```kusto
Usage
| where TimeGenerated > ago(3d)
| summarize GB = sum(Quantity)/1000 by DataType
| top 5 by GB desc
| render barchart
```

**You should see** a bar chart (it may be near-empty in a fresh workspace — that's fine; the query
is what's being validated). If you needed the answer above to write it, do the drill again tomorrow.

## 🚩 Common mistakes

| 🚩 Mistake | Fix |
|---|---|
| `where` *after* `summarize` expecting to filter raw rows | Filter before you aggregate; filter aggregates with a second `where` |
| Forgetting the time filter | Every query starts `where TimeGenerated > ago(...)` — cost and speed |
| `join` without `kind=` | Default is `innerunique` and will surprise you; always state `kind=inner/leftouter` |
| `take` and expecting the "first" rows | `take` is unordered; use `top ... by` for deterministic results |
| `==` for strings ignoring case | `=~` is case-insensitive equals; `has`/`contains` for substrings |

## 🗒️ Log your run

`LOG.md` — paste the query you wrote unaided in Validate.

## 📚 Microsoft Learn

- [KQL quick reference](https://learn.microsoft.com/en-us/kusto/query/kql-quick-reference)
- [Learning path: Write your first queries with KQL](https://learn.microsoft.com/en-us/training/paths/kusto-query-language/)
- [Best practices for KQL queries](https://learn.microsoft.com/en-us/kusto/query/best-practices)

---

<div align="center">
<sub>

[⬅ Prev: 03 · Navigating Sentinel](../03-navigating-sentinel/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 05 · RBAC and roles ➡](../05-rbac-and-roles/README.md)

</sub>
</div>
