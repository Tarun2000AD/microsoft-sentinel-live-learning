# 📜 Changelog

All notable changes to this repository.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] — 2026-09-02

### Added

- The full 63-step line, `00` → `62`, in six phases: Foundations, Data onboarding,
  SIEM rules, Automation & Logic Apps, Threat hunting, Operate at scale.
- Every step folder has a written `README.md` — goal, concepts, portal click-path,
  CLI / IaC, a Validate block with expected output, common mistakes, and Learn links.
- `STEP-LOG-TEMPLATE.md`, `DETECTION-TEMPLATE.md`, `HUNT-TEMPLATE.md` in `_templates/`.
- Structure-check workflow in [`ci/`](ci/README.md) (move it to `.github/workflows/`
  to enable): every step folder has a README, numbering 00–62 is gap-free, no obvious secrets.

### Status

Content is written. Nothing is claimed as *run* — those are the reader's ticks in the
main README, backed by evidence in each step folder.
