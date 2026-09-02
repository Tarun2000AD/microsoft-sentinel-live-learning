<div align="center">

# 🏹 Step 44 · Hunt: identity

### *MFA fatigue, impossible travel, and risky OAuth consent*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~50 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Run three identity hunts against `SigninLogs` / `AuditLogs`, baseline each, simulate the behaviour,
and bookmark the hits.

## 🧠 Why this step

Identity is the most common intrusion path. These three patterns routinely slip past basic rules
because each individual event looks benign.

## ✅ Prerequisites

- [Step 09](../09-microsoft-entra-id/README.md) — `SigninLogs`, `AuditLogs` flowing
- Test users you can push MFA prompts to / grant consent as

## 🧭 The three hunts

### 1️⃣ MFA fatigue (push bombing) — T1621

```kusto
let window = 15m;
SigninLogs
| where TimeGenerated > ago(7d)
| where AuthenticationRequirement == "multiFactorAuthentication"
| extend mfaResult = tostring(Status.additionalDetails)
| summarize
    Prompts = count(),
    Denied = countif(ResultType in (500121, 50074, 50072)),   // MFA denied / not completed
    Approved = countif(ResultType == 0),
    IPs = make_set(IPAddress, 5),
    Apps = make_set(AppDisplayName, 5)
    by UserPrincipalName, bin(TimeGenerated, window)
| where Prompts >= 5 and Denied >= 4 and Approved >= 1     // many denials then one approval
| sort by Prompts desc
```

**Baseline first:** distribution of `Prompts` per user per 15m — most users are 1–2.

### 2️⃣ Impossible travel not flagged by Identity Protection — T1078

```kusto
SigninLogs
| where TimeGenerated > ago(7d) and ResultType == 0
| where isnotempty(LocationDetails.countryOrRegion)
| project TimeGenerated, UserPrincipalName, IPAddress,
          Country = tostring(LocationDetails.countryOrRegion),
          Lat = todouble(LocationDetails.geoCoordinates.latitude),
          Lon = todouble(LocationDetails.geoCoordinates.longitude)
| order by UserPrincipalName asc, TimeGenerated asc
| serialize
| extend PrevTime = prev(TimeGenerated), PrevCountry = prev(Country),
         PrevLat = prev(Lat), PrevLon = prev(Lon), PrevUser = prev(UserPrincipalName)
| where UserPrincipalName == PrevUser and Country != PrevCountry
| extend Hours = (TimeGenerated - PrevTime) / 1h,
         KM = geo_distance_2points(Lon, Lat, PrevLon, PrevLat) / 1000
| extend ImpliedSpeedKmh = KM / Hours
| where Hours < 6 and ImpliedSpeedKmh > 900    // faster than a commercial flight
| project TimeGenerated, UserPrincipalName, PrevCountry, Country, Hours, KM, ImpliedSpeedKmh, IPAddress
```

### 3️⃣ OAuth consent to an over-privileged app by a non-admin — T1528

```kusto
AuditLogs
| where TimeGenerated > ago(30d)
| where OperationName in ("Consent to application", "Add app role assignment grant to user")
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend App = tostring(TargetResources[0].displayName)
| extend Scopes = tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[0].newValue)
| where Scopes has_any ("Mail.Read","Mail.ReadWrite","Files.ReadWrite.All","Directory.Read.All",
                        "User.Read.All","offline_access","full_access_as_user")
| project TimeGenerated, Actor, App, Scopes, Result
```

## 🖱️ Do it

1. Save all three as hunting queries with MITRE tags (step 41).
2. Run each. Record the baseline numbers.
3. **Simulate:**
   - MFA fatigue: trigger 6+ MFA prompts for a test user (repeated sign-in attempts), deny them,
     then approve once.
   - Impossible travel: sign in as a test user, then within an hour sign in again via a VPN exit in
     another country.
   - OAuth: register a test app requesting `Mail.Read` + `offline_access`, consent as a non-admin
     test user.
4. Re-run the hunts; bookmark the hits with notes.

## 🧪 Validate

**You should see** each simulated behaviour appear in its hunt's results and nowhere in the
baseline. For impossible travel, `ImpliedSpeedKmh` should be implausibly high. Bookmark each,
mapping the Account and IP entities, and note which of these deserves promotion to a rule (step 49).

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| MFA fatigue query without the "then approved" clause | Flags every user who fat-fingers MFA twice |
| Impossible travel without `serialize` before `prev()` | `prev()` needs ordered, serialized rows |
| Trusting GeoIP precision | City-level is fuzzy; use country + speed, not street distance |
| OAuth hunt ignoring admin consent | Admin-consented apps are a different (also worth hunting) path |

## 🗒️ Log your run

`LOG.md` + updated `HUNT-IDENTITY-00X.md` files with baselines, hits, and promotion decisions.

## 📚 Microsoft Learn

- [Hunt for security threats — identity queries](https://learn.microsoft.com/en-us/azure/sentinel/hunting)
- [SigninLogs schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/signinlogs)
- [geo_distance_2points() in KQL](https://learn.microsoft.com/en-us/kusto/query/geo-distance-2points-function)
- [Investigate risky OAuth apps](https://learn.microsoft.com/en-us/defender-cloud-apps/investigate-risky-oauth)

---

<div align="center">
<sub>

[⬅ Prev: 43 · Livestream](../43-livestream/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 45 · Hunt: endpoint ➡](../45-hunt-endpoint/README.md)

</sub>
</div>
