# Phase 04 Validation Plan

## Authentication tests

- SSO login succeeds for an enabled workforce user.
- MFA challenge is enforced according to the chosen identity-source design.
- AWS access portal shows the expected assignment.
- CLI SSO returns temporary credentials/session identity.

## Authorization tests

Each role must prove one allowed and one denied operation.

Examples:

```text
Developer
  allowed -> scoped workload read/operate action
  denied  -> IAM administration

Auditor
  allowed -> read-only inspection
  denied  -> create/update/delete

Security
  allowed -> security/audit visibility
  denied  -> infrastructure administration
```

## Lifecycle tests

```text
Joiner  -> identity added -> group assigned -> AWS access appears
Mover   -> group changed -> effective AWS permissions change
Leaver  -> identity disabled/removed -> new AWS access denied
```

## Audit tests

Capture enough evidence to connect a workforce session with AWS activity using CloudTrail/Event History or the appropriate current audit surface.

## Acceptance rule

A configuration screenshot alone is not acceptance. Phase 04 requires observed behavior that matches the intended identity and authorization boundaries.
