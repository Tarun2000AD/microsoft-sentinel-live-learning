<div align="center">

# 🔍 Step 24 · Watchlists

### *Reference data — VIPs, known-bad IPs, asset inventory — inside your rules*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~25 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Two watchlists exist (`VIPUsers`, `HighValueAssets` or `KnownBadIPs`), and a rule that uses one to
**raise severity** or **suppress false positives**.

## 🧠 Why this step

Context you can't get from logs alone — which accounts are executives, which servers are crown
jewels, which IPs your business partners use — lives in a watchlist. Rules join against it to
prioritise or to allowlist.

## ✅ Prerequisites

- [Step 19](../19-write-a-scheduled-rule/README.md) — a rule to enrich
- A CSV to upload (build a tiny synthetic one)

## 🧭 Concepts in 60 seconds

- A watchlist is a named CSV in the workspace, exposed as a function: `_GetWatchlist('VIPUsers')`.
- You choose a **SearchKey** column for fast lookups.
- Small (< ~10k rows) and slow-changing. For big/dynamic data, use a custom table instead.
- **Large watchlists** (preview/GA varies) back onto blob storage for bigger sets.
- Editable in-portal for small fixes; re-upload for bulk.

## 🖱️ Do it — portal

1. Create `vip-users.csv`:

```csv
UserPrincipalName,DisplayName,Department,Tier
ceo@contoso-lab.com,Test CEO,Executive,0
cfo@contoso-lab.com,Test CFO,Executive,0
it-admin@contoso-lab.com,Test Admin,IT,1
```

2. **Microsoft Sentinel → Watchlists → + New** → alias `VIPUsers`, source `vip-users.csv`,
   SearchKey `UserPrincipalName`. Create.
3. Repeat for `known-bad-ips.csv` (`IPAddress,Source,AddedOn`) → alias `KnownBadIPs`, SearchKey
   `IPAddress`.

## 💻 Do it — CLI

```bash
az sentinel watchlist create -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --watchlist-alias VIPUsers --display-name "VIP Users" --provider "lab" --source "vip-users.csv" \
  --items-search-key UserPrincipalName

az sentinel watchlist-item create -g rg-sentinel-lab --workspace-name law-sentinel-lab \
  --watchlist-alias VIPUsers --watchlist-item-id $(uuidgen) \
  --properties '{"UserPrincipalName":"ceo@contoso-lab.com","DisplayName":"Test CEO","Department":"Executive","Tier":"0"}'
```

## 🖱️ Use it in a rule — severity escalation

```kusto
let vips = _GetWatchlist('VIPUsers') | project UserPrincipalName = tolower(UserPrincipalName), Tier;
SigninLogs
| where TimeGenerated > ago(1h) and ResultType != 0
| summarize Failures = count() by UserPrincipalName = tolower(UserPrincipalName), IPAddress
| where Failures >= 5
| join kind=leftouter vips on UserPrincipalName
| extend Severity = iff(isnotempty(Tier), "High", "Medium")
| extend AccountCustomEntity = UserPrincipalName, IPCustomEntity = IPAddress
```

Set **Alert details → Severity** to the `Severity` column so VIP failures become High.

## 🖱️ Use it in a rule — allowlist

```kusto
let partners = _GetWatchlist('KnownBadIPs') | where Source == "partner-allow" | project IPAddress;
CommonSecurityLog
| where TimeGenerated > ago(1h) and DeviceAction == "deny"
| where SourceIP !in (partners)
```

## 🧪 Validate

```kusto
_GetWatchlist('VIPUsers') | count
_GetWatchlist('VIPUsers') | where UserPrincipalName has "ceo"
```

Then fail sign-in 5× for `ceo@contoso-lab.com` and 5× for a non-VIP; confirm the VIP produces a
**High** incident and the non-VIP a **Medium** one.

**You should see** the watchlist function returning your rows, and the severity split in the two
incidents.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Huge/fast-changing data in a watchlist | Slow joins, upload pain — use a custom table |
| Case mismatch on the join key | `tolower()` both sides |
| Wrong SearchKey | Lookups scan the whole list |
| Watchlist as the only allowlist mechanism | Also consider automation-rule suppression (step 35) |

## 🗒️ Log your run

`LOG.md` — the two watchlists, the rule change, and the High-vs-Medium incident evidence. Commit the
**synthetic** CSVs to `artifacts/`.

## 📚 Microsoft Learn

- [Use watchlists in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/watchlists)
- [Build queries and detection rules with watchlists](https://learn.microsoft.com/en-us/azure/sentinel/watchlists-queries)
- [Create large watchlists from a file in Azure Storage](https://learn.microsoft.com/en-us/azure/sentinel/watchlists-create)

---

<div align="center">
<sub>

[⬅ Prev: 23 · NRT rules](../23-nrt-rules/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 25 · MITRE ATT&CK coverage ➡](../25-mitre-attack-coverage/README.md)

</sub>
</div>
