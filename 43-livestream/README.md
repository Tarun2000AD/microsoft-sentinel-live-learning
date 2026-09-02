<div align="center">

# 🏹 Step 43 · Livestream

### *Watch a hunting query fire in near-real-time*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~20 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've run a Livestream session, seen new matches appear as they're ingested, and promoted a live
result to a bookmark.

## 🧠 Why this step

Livestream is for the moment you have a hypothesis about activity *happening right now* — an
in-progress incident, a suspected compromised account — and you want to watch, not wait for a
scheduled rule. It's also how you test a would-be detection before committing it as a rule.

## ✅ Prerequisites

- [Step 41](../41-the-hunting-blade/README.md)
- A way to generate the activity you'll watch for (your lab VM / test accounts)

## 🧭 Concepts in 60 seconds

- Livestream runs a query continuously and streams new results into a live pane.
- It **notifies** you (portal toast) when new results arrive.
- It does **not** create alerts or persist results — it's a live view. Bookmark anything you want
  to keep.
- Good for: active incident monitoring, validating a detection idea, watching a single entity.
- Session stays open while the blade is open; you can save the Livestream and reopen it later.

## 🖱️ Do it — portal

1. **Hunting → Livestream → + New livestream** (or from a query's result → **Livestream**).
2. Query — watch for failed logons to your lab VM in real time:

```kusto
SecurityEvent
| where EventID == 4625
| project TimeGenerated, Computer, TargetUserName, IpAddress, LogonType, SubStatus
```

3. Start it. The pane shows recent matches and a running count.
4. **Generate activity**: fail RDP/SMB logon to `vm-win-lab` a few times.
5. Within ~1–2 minutes, new rows stream in and you get a notification. Select a row → **Add
   bookmark**.
6. **Save** the Livestream as `LS · 4625 on lab VM` so you can reopen it.

## 🧪 Validate

- Before generating activity: the Livestream pane is quiet (baseline).
- After ~5 failed logons: 5 new rows appear within a couple of minutes, with a toast notification.
- Compare the arrival latency to a scheduled rule (up to 1h) and NRT (~1 min) — Livestream is
  roughly NRT-speed but human-watched.
- The bookmark you created from a live row appears in `HuntingBookmark`.

```kusto
HuntingBookmark
| where TimeGenerated > ago(1h) and BookmarkName has "4625"
| project BookmarkName, TimeGenerated, tostring(QueryResultRow.TargetUserName)
```

**You should see** rows appearing live during your simulated activity and a persisted bookmark from
one of them.

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Expecting Livestream results to persist | They don't — bookmark what matters before you close it |
| Using Livestream as a detection | It needs a human watching; promote to NRT/scheduled for real coverage |
| Over-broad query in Livestream | The pane floods; scope to an entity or a tight condition |
| Leaving many Livestreams running | Each is a continuous query; close the ones you're not watching |

## 🗒️ Log your run

`LOG.md` — the latency you observed and the bookmark created from a live row.

## 📚 Microsoft Learn

- [Detect threats by using hunting livestream in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/livestream)

---

<div align="center">
<sub>

[⬅ Prev: 42 · Bookmarks](../42-bookmarks/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 44 · Hunt: identity ➡](../44-hunt-identity/README.md)

</sub>
</div>
