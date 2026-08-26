# Phase 04 Repository Scope

This repository is the implementation record for **MADAR Phase 04 — Enterprise Identity & Workforce Access**.

## This repository owns

- representative corporate Active Directory lab,
- workforce users, groups and role model,
- domain-client join, GPO and local authorization proof,
- CGNAT-aware hybrid network architecture,
- self-managed routed WireGuard connectivity between VMware and AWS,
- AWS Directory Service AD Connector integration,
- dedicated AD Connector service-account handling,
- Amazon WorkSpaces directory registration,
- WorkSpaces target OU and computer-object delegation,
- domain-user authentication to an AWS-managed WorkSpace,
- controlled hybrid-network failure and recovery validation,
- evidence, troubleshooting records, cost guardrails and cleanup,
- Phase 04 technical closeout.

## Architecture boundary

The first design assumed IAM Identity Center, SSO/MFA and permission sets for direct AWS-account workforce access. During execution, the account remained intentionally on the AWS Free Plan and the lab guardrail prohibited upgrading the account or changing Organizations state merely to force that architecture.

Therefore this repository records two things separately:

```text
Validated in the lab
On-prem AD -> WireGuard -> AD Connector -> Amazon WorkSpaces -> domain-user login

Future production extension
On-prem/corporate identity -> IAM Identity Center -> SSO/MFA -> permission sets
```

The future extension is documented as an architectural direction, not falsely presented as implemented.

## This repository does not own

- Phase 03 workload migration implementation,
- Phase 05+ application modernization,
- the full MADAR company roadmap,
- unrelated AWS governance or security work not required by Phase 04.

Those transformation-level concerns remain in `EngMohammedBashir/MADAR-cloud-transformation`.

## Story rule

The VMware Active Directory environment represents a corporate directory that exists before Phase 04 in the MADAR scenario. The lab build is reproducibility work, not a claim that MADAR only acquired employee identities after migrating workloads to AWS.
