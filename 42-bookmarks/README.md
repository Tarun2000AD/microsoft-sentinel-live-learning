<div align="center">

# 🏹 Step 42 · Bookmarks

### *Capture evidence mid-hunt and carry it into an incident*

[![Phase](https://img.shields.io/badge/Phase-Threat hunting-D29922?style=flat-square)](../README.md)
[![Time](https://img.shields.io/badge/Time-~25 min-0078D4?style=flat-square)](#)
[![Cost](https://img.shields.io/badge/Cost-%240-107C10?style=flat-square)](#)

</div>

---

## 🎯 Goal

You've bookmarked interesting rows from a hunt with entity mapping and notes, viewed them in the
`HuntingBookmark` table, and promoted a group of bookmarks into an incident.

## 🧠 Why this step

A hunt produces rows you want to keep — "this logon is weird, come back to it". Bookmarks preserve
the row, the query, the time, your notes and mapped entities, and they can become (or attach to) an
incident so the work isn't lost.

## ✅ Prerequisites

- [Step 41](../41-the-hunting-blade/README.md) — you can run hunting queries

## 🧭 Concepts in 60 seconds

- Bookmark = a saved query result row + metadata (notes, tags, MITRE, mapped entities, the query,
  the time range).
- Stored in the **`HuntingBookmark`** table — queryable like anything else.
- Actions on a bookmark: **add to a new incident**, **add to an existing incident**, **investigate**
  (opens the graph if entities are mapped), **run playbook** (entity trigger).
- Bookmarks show on the **investigation graph** alongside alerts.

## 🖱️ Do it — portal

1. Run a hunting query (e.g. your `HUNT-ENDPOINT-001`, or "Anomalous sign-in location").
2. In the results, select 1–3 rows → **Add bookmark**:
   - Name `LOLBin regsvr32 network arg on vm-win-lab`.
   - Map entities: Account, Host, IP where the columns exist.
   - MITRE: Defense Evasion / T1218.
   - Notes: *"regsvr32 with an http argument at 02:14 UTC — no change ticket, user was offline.
     Needs endpoint review."*
   - Tags: `hunt`, `endpoint`.
3. **Hunting → Bookmarks tab** — your bookmark is listed.
4. Select it (and any related ones) → **Incident actions → Create new incident**. Open the incident
   → the bookmark appears under **Bookmarks**, entities carried over.

## 💻 Do it — query your bookmarks

```kusto
HuntingBookmark
| where TimeGenerated > ago(30d)
| project TimeGenerated, BookmarkName, CreatedBy = tostring(CreatedBy.name),
          Notes, Tags, MitreTactics = Tactics, QueryTime = QueryStartTime,
          Account = tostring(QueryResultRow.AccountName), Host = tostring(QueryResultRow.DeviceName)
| sort by TimeGenerated desc
```

```kusto
// bookmarks not yet attached to an incident — your hunt backlog
HuntingBookmark
| where isempty(IncidentInfo)
| project BookmarkName, TimeGenerated, Notes
```

## 🧪 Validate

**You should see** the bookmark in `HuntingBookmark` with your notes and mapped entities, and — after
promotion — the incident showing the bookmark and its entities on the **Entities** tab and
investigation graph. Open the incident's **Bookmarks** section and confirm the original query is
preserved (click "run query" from the bookmark and get the same rows).

## 🚩 Common mistakes

| 🚩 Mistake | Why it bites |
|---|---|
| Bookmark with no entity mapping | No graph, no correlation, harder to action |
| Bookmark with no notes | Future-you has no idea why it was interesting |
| Bookmarking everything | Bookmarks become noise too — capture decisions, not curiosity |
| Never promoting bookmarks | The hunt findings die in the Bookmarks tab |

## 🗒️ Log your run

`LOG.md` — the `HuntingBookmark` query output (redacted) and the incident it fed. Update the
relevant `HUNT-*.md`'s **Bookmarks created** section.

## 📚 Microsoft Learn

- [Keep track of data during hunting with bookmarks](https://learn.microsoft.com/en-us/azure/sentinel/bookmarks)
- [HuntingBookmark table reference](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/huntingbookmark)

---

<div align="center">
<sub>

[⬅ Prev: 41 · The Hunting blade](../41-the-hunting-blade/README.md) &nbsp;·&nbsp; [🦅 All steps](../README.md) &nbsp;·&nbsp; [Next: 43 · Livestream ➡](../43-livestream/README.md)

</sub>
</div>
