# Local Directory Lab Validation

**Execution date:** 2026-08-24  
**Status:** COMPLETE for the local identity baseline

## Purpose

Before integrating workforce identity with AWS, Phase 04 required a working source-side corporate identity environment. The VMware lab represents MADAR's pre-existing corporate directory and proves that users, groups, policies and client authentication work before any AWS Directory Service resource is provisioned.

## Implemented topology

```text
VMware NAT: 192.168.14.0/24

MADAR-DC01
├── Windows Server 2025
├── IPv4: 192.168.14.10
├── AD DS
├── DNS
└── madar.local
        │
        └── MADAR-CLIENT01
            ├── Windows 11 Pro
            ├── domain joined
            └── domain-user / GPO validation
```

`MADAR-LEGACY-01` remains a separate representative legacy workload and is not the identity source.

## Directory build

Created organizational structure:

- `Management`
- `IT`
- `Finance`
- `HR`
- `Sales`
- `Users`
- `Computers`
- `Groups`

A dedicated `WorkSpaces` OU was added later during the AWS integration stage for cloud-desktop computer objects.

Created Global Security Groups:

- `GG-Management`
- `GG-IT`
- `GG-Finance`
- `GG-HR`
- `GG-Sales`

Synthetic employees:

| User | Department | Group |
|---|---|---|
| Ahmed Ali | Management | `GG-Management` |
| Sara Ibrahim | IT | `GG-IT` |
| Omar Hassan | Finance | `GG-Finance` |
| Noura Saleh | HR | `GG-HR` |
| Khalid Mansour | Sales | `GG-Sales` |

The first user was created manually through Active Directory Users and Computers. The remaining users were created with PowerShell and group membership was independently queried afterward.

## Client validation

`MADAR-CLIENT01` was configured to use `MADAR-DC01` for DNS and successfully joined `madar.local`.

Validation included:

1. DNS connectivity to the domain controller.
2. Successful domain join.
3. Domain-user authentication using the synthetic IT identity.
4. Placement of the client computer in the IT OU.
5. Group Policy refresh and result verification.
6. Domain firewall policy verification.

## Group Policy proof

`GPO-IT-Security` was linked to the IT OU and configured with a Domain firewall baseline.

The client subsequently reported the GPO as applied and the Domain firewall profile as enabled.

This proves the complete path:

```text
AD configuration
      ↓
OU scope
      ↓
GPO link
      ↓
Domain client
      ↓
Policy enforcement
```

## Authorization proof

Two SMB shares were used to make group membership operationally testable:

```text
Sara Ibrahim → GG-IT
      │
      ├── IT share       → ACCESS ALLOWED
      └── Finance share  → ACCESS DENIED
```

The negative result is intentional evidence of authorization working correctly.

## Acceptance result

The local corporate identity source was accepted as the source-side baseline for AWS integration.

```text
Domain Controller      PASS
DNS                    PASS
Users / OUs            PASS
Security Groups        PASS
PowerShell automation  PASS
Domain Join            PASS
Domain Authentication  PASS
GPO Application        PASS
Firewall Enforcement   PASS
Allowed Access Test    PASS
Denied Access Test     PASS
```

## Subsequent AWS integration outcome

This document records the **local baseline checkpoint**. Phase 04 subsequently continued beyond this checkpoint and completed the AWS integration path:

```text
Local AD baseline
      ↓
CGNAT-aware WireGuard hybrid network
      ↓
AWS → on-prem AD/DNS protocol validation
      ↓
AWS Directory Service AD Connector
      ↓
Amazon WorkSpaces
      ↓
WorkSpace computer joined to madar.local
      ↓
madar\sara.ibrahim authenticated
      ↓
VPN failure / recovery validated
      ↓
Temporary AWS resources cleaned up
```

The original IAM Identity Center / permission-set branch was deliberately deferred under the Free Plan/account guardrail and is not claimed as implemented.

For the final Phase 04 state, use [`../CURRENT-STATE.md`](../CURRENT-STATE.md). For the complete rebuild and validation procedure, use [`../runbooks/00-lab-rebuild-and-validation.md`](../runbooks/00-lab-rebuild-and-validation.md).

## Evidence

See [`../evidence/README.md`](../evidence/README.md) for the complete evidence index.
