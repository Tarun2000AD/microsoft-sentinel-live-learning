<div align="center">

# 🔍 Step 20 · Entity mapping & custom details

### *Make incidents correlate, and make them readable at a glance*

[![Phase](https://img.shields.io/badge/Phase-SIEM rules-2F81F7?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Your step-19 rule maps at least three entity types with the right identifiers, surfaces two
**custom details**, and sets **alert details** (dynamic name/description) — and you can see the
effect in the incident view.

## 🧠 Why this step

Entity mapping is what lets Sentinel say "this account appears in 4 other incidents" and what feeds
UEBA, the investigation graph, and playbook automation. Custom details put the *why* on the alert
card so an analyst doesn't have to open Logs. Skipping this is the difference between an incident
queue you can triage and one you can't.

## ✅ Prerequisites

- [Step 19](../19-write-a-scheduled-rule/README.md) — you have a custom rule to improve

## 🧭 Concepts in 60 seconds

| Feature | What it does | Where configured |
|---|---|---|
| **Entity mapping** | Binds query columns to strong identifiers (Account+FullName/Sid/AadUserId, IP+Address, Host+HostName, FileHash, URL, MailMessage…) | Set rule logic → Entity mapping |
| **Custom details** | Key/value pairs from query columns shown on the alert | Set rule logic → Custom details |
| **Alert details** | Override alert Name / Description / Severity / Tactics per row using column values | Set rule logic → Alert details |
| **Dynamic properties** | Same idea, newer, for more alert fields | Alert details |

Each entity type has multiple **identifiers**; map the strongest you have. `Account` with just a
name is weak; `Account` + `AadUserId` or `Sid` correlates far better.

## 🖱️ Do it — portal

1. **Analytics → your DET-IDENTITY-001 rule → Edit → Set rule logic.**
2. **Entity mapping:**
   - `Account` → identifier `Name` = `TargetUserName` (add `NTDomain` if you have it).
   - `IP` → `Address` = `IpAddress`.
   - `Host` → `HostName` = `Computer`.
3. **Custom details:** add `FailCount` → `FailCount`, `FirstFailure` → `FirstFail`,
   `LogonWindow` → `SuccessCount`.
4. **Alert details:** Alert Name format:
   `Brute force hit: {{TargetUserName}} from {{IpAddress}} ({{FailCount}} fails)`.
5. Save. Re-run the step-19 attack simulation.

## 💻 Do it — as the rule's `entityMappings` / `customDetails`

```json
"entityMappings": [
  {"entityType":"Account","fieldMappings":[{"identifier":"Name","columnName":"TargetUserName"}]},
  {"entityType":"IP","fieldMappings":[{"identifier":"Address","columnName":"IpAddress"}]},
  {"entityType":"Host","fieldMappings":[{"identifier":"HostName","columnName":"Computer"}]}
],
"customDetails": { "FailCount":"FailCount", "FirstFailure":"FirstFail" },
"alertDetailsOverride": {
  "alertDisplayNameFormat": "Brute force hit: {{TargetUserName}} from {{IpAddress}} ({{FailCount}} fails)"
}
```

## 🧪 Validate

After the rule fires again:

```kusto
SecurityAlert
| where TimeGenerated > ago(2h) and AlertName has "Brute force hit"
| extend ext = parse_json(ExtendedProperties)
| project TimeGenerated, AlertName, CustomDetails = ext["Custom Details"], Entities
```

In the portal: open the incident → **Entities** tab shows Account / IP / Host as clickable
entities; the alert card title is the dynamic string; **Custom details** shows `FailCount` etc.

**You should see** the alert name now contains the actual username and IP, custom details are
present, and clicking the Account entity opens its entity page. Open a second incident for the same
user and confirm the "related incidents" / entity insights link them.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Mapping only `Account` by name | Weak correlation; add SID / AadUserId when available |
| More than the max entity mappings per rule | Sentinel caps it (currently 10 mappings / 3 identifiers each) — prioritise |
| Custom details with high-cardinality values | The alert gets bloated; keep it to decision-relevant fields |
| Column referenced in mapping not in `project` | Mapping silently yields empty entities |

## 🗒️ Log your run

`LOG.md` — before/after screenshots of the incident entities + alert card; update
`DET-IDENTITY-001.md`.

## 📚 Microsoft Learn

- [Map data fields to entities](https://learn.microsoft.com/en-us/azure/sentinel/map-data-fields-to-entities)
- [Surface custom event details in alerts](https://learn.microsoft.com/en-us/azure/sentinel/surface-custom-details-in-alerts)
- [Customize alert details](https://learn.microsoft.com/en-us/azure/sentinel/customize-alert-details)

---

<div align="center">
<sub>

[⬅ Prev: 19 · Write a scheduled rule](../19-write-a-scheduled-rule/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 21 · Alert & event grouping ➡](../21-alert-and-event-grouping/README.md)

</sub>
</div>
