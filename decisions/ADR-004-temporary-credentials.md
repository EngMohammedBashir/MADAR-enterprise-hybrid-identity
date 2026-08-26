# ADR-005 — Temporary Credentials for Direct AWS Workforce Access

**Status:** Accepted as future production direction  
**Phase 04 implementation:** Not implemented

## 🎯 Context

Long-lived employee-specific AWS access keys create persistent credentials that are harder to govern, rotate and revoke consistently across a workforce.

The long-term MADAR workforce-access model should therefore favor centralized authentication and short-lived AWS sessions.

## 🧭 Decision

For future direct AWS-account access, ordinary employees should authenticate through a centralized workforce identity platform and receive temporary role-based sessions rather than permanent employee-specific AWS access keys.

```text
Corporate identity
      ↓
Central workforce authentication
      ↓
Job-function authorization
      ↓
Temporary AWS session
      ↓
AWS account/resources
```

## 🔐 Why temporary sessions

Temporary credentials reduce credential persistence and align access more closely with centrally managed identity lifecycle and role changes.

They also provide a cleaner operational model for:

- onboarding,
- job-function changes,
- offboarding,
- MFA enforcement,
- session expiration,
- centralized access review.

## 🧪 Phase 04 result

Direct AWS-account console/CLI SSO through IAM Identity Center was **not** implemented in this lab.

The AWS account remained within the agreed Free Plan guardrails, and AWS Organizations/account-plan changes were not forced solely to complete that branch of the original design.

Instead, Phase 04 validated centralized corporate identity consumption through Amazon WorkSpaces:

```text
madar.local user
      ↓
AD Connector
      ↓
Amazon WorkSpaces
      ↓
Successful domain authentication
```

No employee-specific long-lived AWS access key was required for the WorkSpaces authentication proof.

## 🚀 Future validation criteria

When IAM Identity Center organizational prerequisites are intentionally available, validate:

- browser-based workforce SSO,
- MFA,
- group-to-permission-set mapping,
- temporary AWS console sessions,
- AWS CLI SSO,
- `sts get-caller-identity` from temporary credentials,
- session expiration behavior,
- absence of static workforce access keys,
- positive and negative least-privilege tests.

## ⚖️ Consequences

### Benefits

- Reduces persistent workforce credentials.
- Improves centralized lifecycle control.
- Supports role-based authorization and session expiration.
- Provides a natural extension of the group-based identity model already demonstrated in Active Directory.

### Trade-offs

- Requires an intentionally configured central AWS workforce identity architecture.
- Direct AWS SSO availability becomes dependent on that identity platform and its organizational prerequisites.
- This capability must not be claimed as implemented until it has been deployed and validated.

## ✅ Positioning

This ADR records the intended production direction without overstating the completed lab. Phase 04 proves hybrid corporate identity consumption through AD Connector and Amazon WorkSpaces; temporary direct AWS-account workforce sessions remain the next identity-access evolution.