# Runbook — Offboard Workforce User

## Implemented Phase 04 path

1. Disable the employee identity in the corporate `madar.local` directory according to policy.
2. Remove business group memberships if required.
3. Prevent future interactive use of the employee's WorkSpace.
4. Stop or delete the employee's WorkSpace according to retention policy.
5. Confirm the WorkSpace computer object is handled according to device-retirement policy.
6. Verify the employee cannot establish a new domain-authenticated WorkSpaces session.
7. Review recent activity/evidence as required.
8. Record revocation timing.

## Centralized revocation principle

The goal is that the employee identity remains controlled centrally in Active Directory rather than being recreated independently in AWS.

```text
Disable corporate identity
       ↓
Future domain authentication denied
       ↓
Cloud desktop access removed
```

## Guardrail

Do not create employee-specific long-lived AWS access keys as a workaround for offboarding or WorkSpaces access.

## Future production extension

If IAM Identity Center is later used for direct AWS-account access, offboarding must additionally verify that no new SSO session can be created and that old account assignments are removed according to the identity lifecycle policy.
