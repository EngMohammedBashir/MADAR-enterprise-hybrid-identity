# ADR-004 — Temporary Workforce Credentials

**Status:** Accepted as future production direction; not implemented in the Free Plan lab

## Decision

For future direct AWS-account workforce access, ordinary MADAR employees should use temporary centralized sessions rather than employee-specific long-lived AWS access keys.

## Why

Temporary sessions reduce credential persistence, support centralized lifecycle control, and make role changes/offboarding easier to enforce.

## Phase 04 execution result

The lab did **not** implement IAM Identity Center console/CLI SSO because the account was intentionally kept on the AWS Free Plan and Organizations/account-plan changes were not forced merely to complete the original design.

Instead, Phase 04 proved centralized corporate identity consumption through Amazon WorkSpaces:

```text
madar.local user
   ↓
AD Connector
   ↓
AWS WorkSpace
   ↓
Successful domain authentication
```

No employee-specific long-lived AWS access key was required for the WorkSpaces authentication proof.

## Future validation

In an account where IAM Identity Center organizational prerequisites are intentionally available, validate:

- browser SSO,
- MFA,
- temporary AWS sessions,
- AWS CLI SSO,
- caller identity from temporary credentials,
- no static workforce access keys.
