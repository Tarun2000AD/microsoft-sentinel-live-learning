<div align="center">

# 🔐 Security

**This is a lab repository. Read this before you build anything.**

</div>

---

## 🧪 Lab safety

Several steps have you generate attack telemetry on purpose (failed logins, encoded PowerShell,
suspicious role grants). That is only safe under all of these:

| | Rule |
|:--:|---|
| 🔒 | **Isolated subscription.** Never one with anything real in it. |
| 🌐 | **Nothing internet-reachable** while deliberately weakened. A lab VM gets a public IP only behind an NSG locked to your own address, and only while you need it. |
| 👤 | **No real personal data.** Synthetic users, synthetic files, synthetic secrets. |
| ⏱️ | **Time-boxed.** Restore the control the moment the detection or hunt is proven. |
| 🗑️ | **Torn down.** Delete the resource group once the write-up is complete. |
| 💳 | **Budget alert on** before any paid resource is created — [step 06](06-cost-model-and-budget/README.md). |

Simulate activity only against resources you own, in a subscription you control. Nothing here is
intended for use against systems you are not authorised to test.

---

## 🔑 Secrets

Never commit: keys, secrets, connection strings, SAS tokens, certificates, client secrets,
Logic App callback URLs, tenant IDs, subscription IDs, workspace IDs, or unredacted screenshots
containing them. Playbook ARM exports frequently contain callback URLs and connection resource
IDs — scrub them.

If a secret does get committed:

1. Rotate it immediately — assume it is public the moment it lands.
2. Then worry about rewriting history.
3. Record what happened in the relevant step log. It is a real incident-response exercise.

`.gitignore` blocks the common offenders, but it is not a substitute for looking at your diff.

---

## 📨 Reporting

Found something here that does more than it says, or an unsafe instruction? Open an issue, or
contact the repository owner directly if public disclosure would be the wrong move.
