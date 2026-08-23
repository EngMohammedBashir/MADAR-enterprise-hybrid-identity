# Phase 04 Repository Scope

This repository is the implementation record for **MADAR Phase 04 — Enterprise Identity & Workforce Access**.

## This repository owns

- representative corporate Active Directory lab,
- workforce users, groups and role model,
- current-state identity assessment,
- target identity architecture,
- AWS IAM Identity Center configuration evidence,
- permission-set design,
- SSO and MFA validation,
- temporary credential / CLI SSO testing,
- positive and negative authorization tests,
- onboarding / role-change / offboarding tests,
- CloudTrail/audit evidence,
- cost guardrails and cleanup,
- Phase 04 technical closeout.

## This repository does not own

- Phase 03 workload migration implementation,
- Phase 05+ application modernization,
- the full MADAR company roadmap,
- unrelated AWS governance or security work not required by Phase 04.

Those transformation-level concerns remain in `EngMohammedBashir/MADAR-cloud-transformation`.

## Story rule

The VMware Active Directory environment built here represents a corporate directory that exists before Phase 04 in the MADAR scenario. The lab build is reproducibility work, not a claim that MADAR only acquired employee identities after migrating workloads to AWS.
