# Current-State Identity Assessment

## Scenario

MADAR already has corporate employee identities outside AWS. Phase 04 represents that system with a VMware-hosted Windows Server Active Directory environment and then proves that AWS can consume that identity source without duplicating users.

## Original risk

Without centralized workforce identity, cloud-hosted employee access tends toward duplicate identities, manual credential lifecycle and poor revocation.

```text
employee
   ↓
separate cloud identity
   ↓
password / static credential
   ↓
manual permission lifecycle
```

That creates credential sprawl, difficult offboarding, inconsistent least privilege and weak audit clarity.

## Implemented transition

```text
Corporate Active Directory
      ↓
Routed WireGuard hybrid network
      ↓
AWS Directory Service AD Connector
      ↓
Amazon WorkSpaces Personal
      ↓
Domain-joined cloud desktop
      ↓
Corporate user authentication
```

The synthetic IT user `sara.ibrahim` was authenticated successfully to an AWS WorkSpace using the existing `madar.local` identity.

## Why this matters

The WorkSpace does not own a separate employee identity. It depends on the corporate directory for authentication. This proves a real hybrid identity dependency rather than simply creating another Windows VM in AWS.

## Local baseline retained

`MADAR-CLIENT01` remains part of the project because it proves the source directory before AWS integration:

- domain join,
- user login,
- GPO enforcement,
- Domain firewall policy,
- allowed IT-share access,
- denied Finance-share access.

The WorkSpace proves the cloud side of the same corporate identity story.

## Scope boundary

This phase focuses on human workforce identity and access. Application/service identities, workload IAM roles and broad multi-account governance belong to their own phases unless Phase 04 needs them for a specific proof.

IAM Identity Center / SSO remains a future extension for an account where the required organizational prerequisites are intentionally available; it was not falsely marked as implemented in this Free Plan lab.
