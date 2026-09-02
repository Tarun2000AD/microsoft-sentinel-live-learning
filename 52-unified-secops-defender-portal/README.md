<div align="center">

# 🛰️ Step 52 · Unified SecOps in the Defender portal

### *Run Sentinel from security.microsoft.com*

[![Phase](https://img.shields.io/badge/Phase-Operate at scale-8957E5?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~30 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

Your Sentinel workspace is onboarded to the **Microsoft Defender portal**, and you've triaged an
incident, run a hunt, and edited a rule there instead of in the Azure portal.

## 🧠 Why this step

Microsoft's direction is one SOC pane: Defender XDR + Sentinel unified at
[security.microsoft.com](https://security.microsoft.com). Incidents from both correlate
automatically, advanced hunting spans XDR + Sentinel tables, and Copilot for Security plugs in
here. The Azure portal Sentinel experience still works but is no longer where new capability lands.

## ✅ Prerequisites

- [Step 10](../10-defender-xdr/README.md) — Defender XDR connected (recommended before unifying)
- **Global Administrator** or **Security Administrator**
- One Log Analytics workspace with Sentinel (multi-workspace unification has extra rules — step 53)

## 🧭 What changes when you onboard

| Area | Azure portal | Defender portal (unified) |
|---|---|---|
| Incident queue | Sentinel Incidents | **Incidents** — XDR + Sentinel merged & correlated |
| Hunting | Sentinel Hunting + Logs | **Advanced hunting** — Sentinel tables + `Device*`/`Email*`/`Identity*` in one schema |
| Analytics rules | Sentinel Analytics | **Detection rules → Scheduled** (same rules, new home) |
| Automation | Sentinel Automation | Same automation rules & playbooks, surfaced in the portal |
| Entities | Entity pages | Unified entity pages (device + identity + Sentinel context) |
| Billing / data connectors / workspace settings | **still Azure portal** | — |

Onboarding is **non-destructive and reversible**; your rules, incidents and data are untouched.

## 🖱️ Do it

1. Go to [security.microsoft.com](https://security.microsoft.com) → **System → Settings →
   Microsoft Sentinel** (or the **Sentinel** node in the left nav) → **Onboard / connect a
   workspace**.
2. Select `law-sentinel-lab` → confirm the primary workspace → onboard. Wait a few minutes.
3. Explore:
   - **Incidents** — your DET-IDENTITY-001 incidents now appear alongside any XDR incidents,
     correlated where they share entities.
   - **Hunting → Advanced hunting** — run a query joining a Sentinel table to an XDR table:

```kql
SigninLogs
| where TimeGenerated > ago(1d) and ResultType == 0
| join kind=inner (
    DeviceLogonEvents | where TimeGenerated > ago(1d)
  ) on $left.UserPrincipalName == $right.AccountUpn
| project TimeGenerated, UserPrincipalName, DeviceName, IPAddress, LogonType
```

   - **Detection rules** — open DET-IDENTITY-001, make a small edit, save. Confirm it reflects back
     in the Azure portal.

## 🧪 Validate

- **Incidents** in the Defender portal lists your Sentinel incidents (filter Source = Microsoft
  Sentinel) and any XDR ones.
- The cross-product advanced-hunting query above returns rows (or runs clean if no overlap yet).
- An edit made in the Defender portal appears in the Azure portal's Analytics blade → single source
  of truth, two front doors.

```kusto
SecurityIncident
| where TimeGenerated > ago(7d)
| summarize count() by ProviderName
```

**You should see** incidents from both `Microsoft Sentinel` and `Microsoft 365 Defender` /
`Microsoft Defender XDR` providers in one queue.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Expecting billing/connectors to move | Those stay in the Azure portal |
| Onboarding before connecting XDR | You get the new UI but not the correlation benefit |
| Assuming it's irreversible | It's reversible; but plan the change with your team |
| Running rules in two places mentally | It's one rule engine — pick a primary UI for your team |

## 🗒️ Log your run

`LOG.md` — the unified incident queue screenshot (redacted), the cross-product hunt result, and the
round-trip edit proof.

## 📚 Microsoft Learn

- [Microsoft Sentinel in the Microsoft Defender portal](https://learn.microsoft.com/en-us/azure/sentinel/microsoft-sentinel-defender-portal)
- [Connect Microsoft Sentinel to the Defender portal](https://learn.microsoft.com/en-us/unified-secops-platform/microsoft-sentinel-onboard)
- [Advanced hunting in the unified portal](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview)

---

<div align="center">
<sub>

[⬅ Prev: 51 · UEBA & entity behavior](../51-ueba-and-entity-behavior/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 53 · Workspace architecture ➡](../53-workspace-architecture/README.md)

</sub>
</div>
