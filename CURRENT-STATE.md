# Phase 04 — Current State

**Status:** PLANNED / READY FOR EXECUTION  
**Date:** 2026-08-23  
**Primary theme:** Enterprise workforce identity and AWS access governance

## Starting point

Phase 03 is complete. MADAR's representative application/data migration has been accepted and temporary migration infrastructure cleaned up.

The next problem is human access:

```text
AWS usage grows
      ↓
more employees need access
      ↓
individual IAM users / long-lived keys do not scale safely
      ↓
centralized workforce identity is required
```

## Scenario baseline

MADAR is assumed to already have a corporate directory for employees. Phase 04 will build a small VMware-hosted Windows Server / Active Directory environment as the **reproducible representation** of that pre-existing directory.

The directory is not a second Phase 03 workload and will not be migrated as EC2 merely for demonstration. Its purpose is to act as the corporate identity source for the workforce-access design.

## Planned implementation gates

```text
Gate 1  Business problem / current-state identity assessment
Gate 2  VMware Windows Server + AD DS lab baseline
Gate 3  workforce users/groups and permission matrix
Gate 4  supported AWS identity-integration architecture confirmed
Gate 5  IAM Identity Center / SSO / MFA configuration
Gate 6  permission sets and account assignments
Gate 7  positive + negative least-privilege tests
Gate 8  console SSO + CLI SSO temporary credentials
Gate 9  onboarding / role change / offboarding tests
Gate 10 audit evidence
Gate 11 cost + cleanup audit
```

## Cost hold point

No paid AWS directory resource is approved yet. Before creation, Phase 04 must verify the exact supported integration path, current `us-east-1` pricing, and expected test-window cost.

## Next action

Begin with the local corporate-directory lab and the identity/permission design. AWS integration starts only after the source identity model is documented and the supported target architecture is confirmed.
