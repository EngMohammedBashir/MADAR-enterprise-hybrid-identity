# Phase 04 — Current State

**Status:** ACTIVE — LOCAL IDENTITY LAB VALIDATED / AWS INTEGRATION NEXT  
**Date:** 2026-08-24  
**Primary theme:** Enterprise workforce identity and AWS access governance

## Starting point

Phase 03 is complete and cleaned up. Phase 04 is now actively executing the workforce-identity design that follows the workload migration.

## Completed local identity baseline

The representative corporate directory is no longer only planned; it has been built and validated in VMware.

```text
MADAR-DC01
├── Windows Server 2025
├── static IPv4: 192.168.14.10
├── AD DS + DNS
├── domain: madar.local
├── department OUs
├── Global Security Groups
└── GPO-IT-Security

MADAR-CLIENT01
├── Windows 11 Pro
├── joined to madar.local
├── domain authentication validated
└── GPO / firewall enforcement validated
```

Synthetic workforce identities:

| Department | User | Group |
|---|---|---|
| Management | Ahmed Ali (`ahmed.ali`) | `GG-Management` |
| IT | Sara Ibrahim (`sara.ibrahim`) | `GG-IT` |
| Finance | Omar Hassan (`omar.hassan`) | `GG-Finance` |
| HR | Noura Saleh (`noura.saleh`) | `GG-HR` |
| Sales | Khalid Mansour (`khalid.mansour`) | `GG-Sales` |

## Local validation completed

- domain-controller role and `madar.local` membership verified,
- users distributed into departmental OUs,
- group membership verified using PowerShell,
- Windows 11 client joined to the domain,
- domain-user login validated,
- `GPO-IT-Security` confirmed as applied,
- Domain firewall profile confirmed enabled,
- IT share access succeeded for the IT user,
- Finance share access was denied for the same IT user as an intentional authorization boundary.

This gives Phase 04 a working source-side identity system before AWS integration begins.

## Current execution gates

```text
Gate 1  Business problem / current-state identity assessment       COMPLETE
Gate 2  VMware Windows Server + AD DS lab baseline                COMPLETE
Gate 3  Workforce users/groups + local authorization validation   COMPLETE
Gate 4  Supported AWS identity-integration architecture           NEXT
Gate 5  IAM Identity Center / SSO / MFA configuration              PENDING
Gate 6  Permission sets and account assignments                    PENDING
Gate 7  Positive + negative AWS least-privilege tests              PENDING
Gate 8  Console SSO + CLI SSO temporary credentials                PENDING
Gate 9  Onboarding / role change / offboarding tests               PENDING
Gate 10 Audit evidence                                              PENDING
Gate 11 Cost + cleanup audit                                        PENDING
```

## Cost hold point

No paid AWS directory resource has been created. Before the next gate, Phase 04 must verify the exact supported integration path, current `us-east-1` pricing, and expected test-window cost.

## Runtime constraint

The local lab runs on a memory-constrained workstation. `MADAR-DC01`, `MADAR-CLIENT01`, and `MADAR-LEGACY-01` are therefore powered on only when required for a test; all three do not need to run simultaneously.

## Next action

Move to the AWS integration design: confirm the currently supported Active Directory → AWS identity path and its cost, then provision only the minimum AWS resources required for IAM Identity Center, SSO/MFA and least-privilege workforce validation.
