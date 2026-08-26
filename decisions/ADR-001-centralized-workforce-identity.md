# ADR-001 — Centralize Workforce Identity

**Status:** Accepted and partially implemented in Phase 04

## Context

MADAR's AWS footprint now supports migrated and cloud-native workloads. Managing ordinary employee access with unrelated cloud identities and long-lived credentials would not scale safely.

## Decision

Keep the existing corporate Active Directory as the source identity authority and prove that AWS can consume that identity centrally rather than creating duplicate users.

The Phase 04 lab validates this through:

```text
madar.local
   ↓
WireGuard hybrid connectivity
   ↓
AWS Directory Service AD Connector
   ↓
Amazon WorkSpaces
   ↓
Corporate user authentication
```

## Original SSO design

IAM Identity Center remains the preferred future pattern for direct AWS-account workforce SSO, temporary sessions and permission sets. It was not forced in this Free Plan lab because the account/Organizations prerequisites were outside the agreed guardrails.

## Consequences

- employee identity remains centralized in the corporate directory,
- AWS can consume the directory without duplicating the employee account,
- WorkSpaces proves end-to-end hybrid authentication,
- IAM users are still not the preferred future workforce pattern,
- direct AWS-account SSO remains a production extension rather than a falsely claimed lab result.
