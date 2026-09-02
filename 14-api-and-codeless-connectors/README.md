<div align="center">

# 📥 Step 14 · API & codeless connectors

### *Pull a third-party feed with no agent and no code*

[![Phase](https://img.shields.io/badge/Phase-Data onboarding-1F6FEB?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~40 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-tiny-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You ingest a REST-API data source into a custom table using either the **Codeless Connector
Platform (CCP)** or a **Logic App poller**, and you understand when each fits.

## 🧠 Why this step

Many SaaS security products (identity providers, email security, DNS, CASBs) have no push
integration — you poll their API. Three routes: a vendor-published **CCP connector** in the Content
hub (best), a **CCP connector you define** in JSON (schedule, auth, pagination, DCR mapping), or a
**Logic App** on a recurrence trigger calling the API and posting to the Logs Ingestion API
(most flexible, has a per-action cost).

## ✅ Prerequisites

- [Step 13](../13-custom-logs-and-dcr-transformations/README.md) — you understand DCR-based custom tables
- An API to pull. Free options for the lab: GitHub audit log API, a threat-feed JSON, `ip-api`, or
  the AbuseIPDB free tier.

## 🧭 Concepts in 60 seconds

| Route | Auth it supports | Schedule | Cost | Use when |
|---|---|---|---|---|
| **CCP — vendor connector** | OAuth2, API key, Basic | Built-in | Ingestion only | A published connector exists |
| **CCP — your own JSON** | OAuth2, API key, AWS, GCP, Basic | `queryWindowInMin` | Ingestion only | No published connector, standard REST + pagination |
| **Logic App poller** | anything (custom code, HMAC…) | Recurrence trigger | Ingestion + Logic App actions | Weird auth, transforms, or you need branching |

## 🖱️ Do it — CCP connector (JSON), the modern route

1. Author a connector definition (`RestApiPoller` kind) — auth block, request block (URL, method,
   query params, pagination), response block (events JSON path), and a DCR + custom table.
2. Deploy it via ARM to your workspace (`Microsoft.SecurityInsights/dataConnectors` +
   `dataConnectorDefinitions`).
3. It appears in **Data connectors** like any built-in one → **Connect** → supply the API key.

Minimal shape:

```json
{
  "kind": "RestApiPoller",
  "properties": {
    "connectorDefinitionName": "AbuseIPDBBlacklist",
    "dataType": "AbuseIPDB_CL",
    "auth": { "type": "APIKey", "apiKeyName": "Key", "apiKeyIdentifier": "Key" },
    "request": {
      "apiEndpoint": "https://api.abuseipdb.com/api/v2/blacklist",
      "httpMethod": "GET",
      "queryParameters": { "confidenceMinimum": "90" },
      "headers": { "Accept": "application/json" },
      "queryWindowInMin": 1440
    },
    "response": { "eventsJsonPaths": ["$.data"] },
    "dcrConfig": { "dataCollectionEndpoint": "<DCE>", "dataCollectionRuleImmutableId": "<id>", "streamName": "Custom-AbuseIPDB_CL" }
  }
}
```

## 💻 Do it — Logic App poller (fallback)

```
Recurrence (every 6h)
  → HTTP GET https://api.example.com/events?since=@{addHours(utcNow(),-6)}
      Authentication: API key header
  → Parse JSON
  → For each item → HTTP POST to the Logs Ingestion API endpoint of your DCR
```

Grant the Logic App's managed identity **Monitoring Metrics Publisher** on the DCR (step 32 covers
managed identity properly).

## 🧪 Validate

```kusto
AbuseIPDB_CL   // or your table
| where TimeGenerated > ago(1d)
| summarize rows = count(), distinctIps = dcount(column_ifexists("ipAddress_s","IPAddress"))
```

```kusto
// prove the poll is recurring, not one-off
AbuseIPDB_CL
| summarize count() by bin(TimeGenerated, 6h)
| render columnchart
```

**You should see** rows arriving on the polling cadence (a bar every interval), not a single spike.
For a Logic App route, also check **Logic App → Runs history** shows repeated `Succeeded` runs.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| No pagination handling | You silently ingest only the first page every poll |
| No de-dupe / cursor | Overlapping windows re-ingest the same records — cost + noisy detections |
| Logic App with hundreds of per-item POSTs | Action count adds up; batch the POST body |
| Storing the API key in a Logic App parameter | Use Key Vault + managed identity |

## 🗒️ Log your run

`LOG.md` — which route, the source API, the polling window, and the recurrence-proof chart.

## 📚 Microsoft Learn

- [Create a codeless connector for Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/create-codeless-connector)
- [Codeless Connector Platform reference](https://learn.microsoft.com/en-us/azure/sentinel/data-connector-connection-rules-reference)
- [Use Logic Apps to ingest data into Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/create-custom-connector#connect-with-logic-apps)

---

<div align="center">
<sub>

[⬅ Prev: 13 · Custom logs + DCR transformations](../13-custom-logs-and-dcr-transformations/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 15 · Ingestion health & validation ➡](../15-ingestion-health-and-validation/README.md)

</sub>
</div>
