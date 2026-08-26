# ADR-002 — Reuse the Existing Corporate Identity Source

**Status:** Accepted and implemented

## Context

The MADAR scenario already contains employee identities before Phase 04. Creating a second unrelated user directory purely for AWS would weaken the enterprise story and create duplicate lifecycle management.

## Decision

Represent the corporate identity source with a VMware-hosted Microsoft Active Directory lab and integrate that source with AWS through a routed WireGuard hybrid path and AWS Directory Service AD Connector.

The directory was then consumed by Amazon WorkSpaces Personal to prove a real domain-user authentication flow.

## Implemented path

```text
MADAR-DC01 / madar.local
        ↓
MADAR-WG01
        ↓
WireGuard
        ↓
AWS WG-HUB
        ↓
AD Connector d-90667da553
        ↓
Amazon WorkSpaces
        ↓
madar\sara.ibrahim login
```

## Guardrail

The account remained on the AWS Free Plan. No account-plan upgrade or Organizations change was performed merely to force an IAM Identity Center branch.

## Consequences

- the project demonstrates real directory integration rather than duplicate users,
- the same employee identity was used locally and in an AWS-managed cloud desktop,
- the local VM can be retained powered off for later phases,
- paid integration infrastructure can be removed after evidence is complete,
- IAM Identity Center remains a future production extension rather than an implemented Phase 04 claim.
