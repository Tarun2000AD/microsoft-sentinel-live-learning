<div align="center">

# 🏹 Step 51 · UEBA & entity behavior

### *Enable UEBA and hunt on the behavior tables*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min setup + days to baseline-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-UEBA tables ingestion (free-ish) + source data-E3B341?style=flat-square)](#)

</div>

---

## 🎯 Goal

UEBA is enabled, the `BehaviorAnalytics` / `IdentityInfo` tables are populating, and you've written
a hunt that uses a UEBA anomaly score.

## 🧠 Why this step

UEBA baselines every user and host over time and scores each action for how unusual it is *for that
entity and its peers*. It turns "a login from Brazil" into "a login from Brazil, which this account
has never done, on a device it's never used, at an hour it never works" — one enriched record.

## ✅ Prerequisites

- [Step 09](../09-microsoft-entra-id/README.md) — `SigninLogs`, `AuditLogs` (UEBA's primary inputs)
- Optionally `SecurityEvent`, `AzureActivity` for host/Azure entity behavior
- A few days of data so baselines form — enable now, hunt later

## 🧭 What UEBA gives you

| Table | Contents |
|---|---|
| `BehaviorAnalytics` | Per-action anomaly records: `InvestigationPriority` score, `ActivityType`, `UsersInsights`, `DevicesInsights`, `ActivityInsights` (first-time flags, peer comparisons) |
| `IdentityInfo` | Enriched directory data per user: groups, roles, manager, MFA, account age — join target for any identity hunt |
| `UserPeerAnalytics` | Who each user's behavioral peers are |
| `Anomalies` | Output of the built-in anomaly rules (overlaps, step 59) |

## 🖱️ Do it — enable

1. **Microsoft Sentinel → Settings → Settings tab → User and Entity Behavior Analytics → Configure.**
2. Turn **On**. Select data sources: **Audit Logs, Sign-in Logs, Azure Activity, Security Events**
   (whatever you have connected).
3. Save. Baselines take **1–14 days**; `IdentityInfo` populates within hours.

## 💻 Do it — hunt once data exists

```kusto
// high-priority anomalous identity activity
BehaviorAnalytics
| where TimeGenerated > ago(7d)
| where InvestigationPriority >= 5
| project TimeGenerated, UserPrincipalName, ActivityType, ActionType,
          Priority = InvestigationPriority,
          FirstTimeUserConnectedFromCountry = tostring(ActivityInsights.FirstTimeUserConnectedFromCountry),
          FirstTimeUserUsedApp = tostring(ActivityInsights.FirstTimeUserConnectedViaISP),
          UncommonForUser = tostring(ActivityInsights.ActionUncommonlyPerformedByUser),
          SourceIP, SourceDevice
| sort by Priority desc
```

```kusto
// enrich ANY identity hit with directory context
let risky = BehaviorAnalytics
    | where TimeGenerated > ago(1d) and InvestigationPriority >= 4
    | distinct UserPrincipalName;
IdentityInfo
| where TimeGenerated > ago(14d)
| where UserPrincipalName in (risky)
| summarize arg_max(TimeGenerated, *) by UserPrincipalName
| project UserPrincipalName, JobTitle, Department, AssignedRoles, GroupMembership,
          IsAccountEnabled, UserAccountControl, AccountCreationTime
```

```kusto
// blast radius: did an anomalous user act outside their peer group?
BehaviorAnalytics
| where TimeGenerated > ago(7d)
| where tostring(UsersInsights.BlastRadius) == "High"
   or tostring(ActivityInsights.ActionUncommonlyPerformedByPeers) == "True"
| project TimeGenerated, UserPrincipalName, ActivityType, InvestigationPriority, UsersInsights, ActivityInsights
```

## 🧪 Validate

- Within hours: `IdentityInfo | count` > 0, one row per directory user.
- After a few days: `BehaviorAnalytics | summarize count() by ActivityType` shows records.
- Generate an anomaly: sign in as a stable test user from a **new country + new device** after it
  has a baseline; within a day it should appear in `BehaviorAnalytics` with a non-trivial
  `InvestigationPriority` and `FirstTimeUserConnectedFromCountry = true`.
- The user's **Entity page** (Sentinel → Entity behavior → search the user) shows the timeline,
  peers, and the anomaly.

**You should see** `IdentityInfo` populated quickly, `BehaviorAnalytics` populated after
baselining, and your deliberate new-country sign-in scored as anomalous.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Expecting anomalies immediately | Baselining takes days; only `IdentityInfo` is fast |
| Treating `InvestigationPriority` as a verdict | It's a triage score — investigate, don't auto-act |
| Not joining `IdentityInfo` in identity hunts | You miss "this is a Global Admin" context for free |
| Enabling UEBA with only one thin source | Poorer baselines; feed it sign-ins + audit + activity |

## 🗒️ Log your run

`LOG.md` — the enable date, `IdentityInfo` row count, and (later) the anomaly you generated with its
score.

## 📚 Microsoft Learn

- [Identify advanced threats with UEBA in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/identify-threats-with-entity-behavior-analytics)
- [Enable UEBA](https://learn.microsoft.com/en-us/azure/sentinel/enable-entity-behavior-analytics)
- [BehaviorAnalytics table reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/behavioranalytics)
- [IdentityInfo table reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/identityinfo)

---

<div align="center">
<sub>

[⬅ Prev: 50 · Notebooks & MSTICPy](../50-notebooks-and-msticpy/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 52 · Unified SecOps (Defender portal) ➡](../52-unified-secops-defender-portal/README.md)

</sub>
</div>
