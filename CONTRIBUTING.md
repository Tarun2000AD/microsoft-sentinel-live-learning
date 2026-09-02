<div align="center">

# 🤝 Contributing

**Rules for writing work into this repository — including your own.**

</div>

---

## 📏 The one rule that matters

> **Never fabricate completion, screenshots, query output or findings.**

Everything else is style. This one is not.

---

## 📝 Logging a step you ran

1. Copy [`_templates/STEP-LOG-TEMPLATE.md`](_templates/STEP-LOG-TEMPLATE.md) into the step folder.
2. Name it `LOG.md` (or `LOG-<yyyy-mm-dd>.md` if you re-run it).
3. Fill in every section. If a section does not apply, write *why* rather than deleting it.
4. Record what failed. Half the value is the part that did not work first time.
5. Tick the box in the main [README](README.md) only after the evidence is in the folder.

## 🖼️ Evidence

| Kind | Where | Rules |
|---|---|---|
| Screenshots | `<step>/evidence/` | Redact tenant IDs, subscription IDs, UPNs, IPs, workspace IDs |
| CLI / query output | fenced ` ```text ` block in `LOG.md` | Redact the same; keep the command |
| Config | `<step>/artifacts/` as `.json` / `.bicep` / `.kql` | No secrets, no callback URLs |
| KQL | fenced ` ```kusto ` block | Must run as written |

## 🗂️ Naming

| Thing | Pattern | Example |
|---|---|---|
| Step log | `LOG.md` | `19-write-a-scheduled-rule/LOG.md` |
| Detection | `DET-<DOMAIN>-<NNN>.md` | `DET-IDENTITY-002.md` |
| Hunt | `HUNT-<DOMAIN>-<NNN>.md` | `HUNT-ENDPOINT-001.md` |
| Query artifact | `<slug>.kql` | `brute-force-signin.kql` |
| Folder | lower-kebab-case | `entity-mapping-and-custom-details/` |

## 🔗 Links

Every step links to **Microsoft Learn**, not to a copy of the docs. Write locally only what Learn
cannot give you: your order, your lab, your detections, your evidence, your decisions.

## ✅ Before committing

- [ ] No secrets, keys, tokens, callback URLs — in files *or* screenshots
- [ ] No tenant / subscription / workspace / object IDs left visible
- [ ] Every claim in the log is something that actually happened
- [ ] KQL runs as written
- [ ] Checkboxes reflect reality
