# Phase 04 Business Case

## Problem

MADAR has completed its first three AWS transformation phases. As employees increasingly need cloud-hosted work environments and access to AWS-hosted systems, keeping identity separate from the existing corporate directory would create duplicate lifecycle management, inconsistent access control and weak offboarding.

## Business requirement

Reuse the existing corporate Active Directory identity source and prove that an AWS-managed workforce environment can authenticate a MADAR employee without creating a second unrelated user account.

## Why now

Phase 03 moved a representative legacy workload into AWS. The next operational risk is no longer only infrastructure or data migration; it is **who can use the cloud environment, how that identity is verified, and whether access still depends on the corporate directory and network path.**

## Implemented lab outcome

The original target included IAM Identity Center / SSO. That branch was not forced because the account was intentionally kept on the AWS Free Plan and the lab must not upgrade the account or change Organizations state merely to satisfy an initial architecture diagram.

The implemented and verified outcome became:

```text
Corporate AD user
      ↓
On-premises madar.local
      ↓
WireGuard hybrid connectivity
      ↓
AWS Directory Service AD Connector
      ↓
Amazon WorkSpaces Personal
      ↓
Domain-joined cloud desktop
      ↓
Successful employee authentication
```

## Success criteria

- corporate identity remains centralized in `madar.local`,
- AWS reaches the private on-prem directory through a routed encrypted tunnel,
- AD Connector reaches `Active`,
- an AWS WorkSpace is domain joined to `madar.local`,
- its computer object is created in the intended AD OU,
- `sara.ibrahim` signs in using the existing corporate AD identity,
- a controlled VPN outage breaks the expected dependency,
- restoring the tunnel restores connectivity,
- temporary paid resources are cost-controlled and cleaned up.
