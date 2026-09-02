<div align="center">

# 🗺️ Roadmap

**The order, and where it currently stands.**

</div>

---

> [!NOTE]
> Status reflects what is actually written in each step folder, not a plan. A step is `written`
> when its `README.md` has all sections filled with real, runnable content. It is `done` (your tick
> in the main [README](README.md)) only when **you** have run it and the Validate block passed in
> your workspace.

---

## The six phases

| Phase | Steps | Theme | Content status |
|---|---|---|---|
| 🧱 Foundations | `00`–`06` | Subscription, workspace, roles, KQL, cost | `written` |
| 📥 Data onboarding | `07`–`16` | Connectors, agents, custom logs, health | `written` |
| 🔍 SIEM rules | `17`–`28` | Analytics rules end to end | `written` |
| 🔄 Automation & Logic Apps | `29`–`39` | Automation rules, playbooks, response | `written` |
| 🏹 Threat hunting | `40`–`51` | Hypotheses, hunting blade, five hunts, UEBA | `written` |
| 🛰️ Operate at scale | `52`–`62` | Unified SecOps, CI/CD, cost, TI, capstone | `written` |

---

## 🏁 Milestones — evidence it actually happened

| | Milestone | Proof |
|:--:|---|---|
| <sub>&#9744;</sub> | Sentinel enabled inside budget | Cost export + a budget alert that fired |
| <sub>&#9744;</sub> | First three connectors live | `Heartbeat` / `SigninLogs` / `AzureActivity` rows |
| <sub>&#9744;</sub> | First detection you wrote fires | The KQL, the incident, the write-up (step `19`) |
| <sub>&#9744;</sub> | First automated response | Playbook run history (step `34`) |
| <sub>&#9744;</sub> | Ten analytics rules live and tuned | Rule list + tuning notes each |
| <sub>&#9744;</sub> | First hunt turned into a rule | Bookmark → rule diff (step `49`) |
| <sub>&#9744;</sub> | All content deployable from Git | A green pipeline run (step `55`) |
| <sub>&#9744;</sub> | Capstone written | `62-capstone/` with a full incident narrative |

---

## ⚠️ Certification caveat

**AZ-500 retires 31 August 2026.** SC-200 (Security Operations Analyst) is the Sentinel-centric
exam and remains active. This path targets the underlying SOC engineering competency; verify current
exam status on [Microsoft Learn](https://learn.microsoft.com/en-us/credentials/certifications/)
before planning around any exam.

---

## 🔭 Not in scope (on purpose)

- **Deep Defender XDR** beyond the Sentinel integration — see the companion path's Defender modules.
- **Full KQL language reference** — step `04` is a survival kit, not a course. Learn links go deeper.
- **Terraform provider** for Sentinel — the as-code steps use ARM/Bicep + the Sentinel repos feature,
  which is the Microsoft-supported route. Terraform notes are mentioned where relevant.
