# ADR-003 — Group-Based Authorization

**Status:** Accepted  
**Local authorization:** Implemented and validated  
**Direct AWS permission-set mapping:** Future extension

## 🎯 Context

Assigning access directly to individual employees creates brittle authorization and makes onboarding, transfers and offboarding difficult to operate consistently.

The MADAR identity model therefore requires access to follow job function and group membership rather than one-off per-user permissions.

## 🧭 Decision

Use Active Directory security groups as the primary authorization abstraction for workforce access.

```text
Employee
   ↓
Department / job function
   ↓
Security group
   ↓
Authorized resource
```

## 🏢 Implemented directory model

The source Active Directory contains departmental Global Security Groups:

- `GG-Management`
- `GG-IT`
- `GG-Finance`
- `GG-HR`
- `GG-Sales`

This keeps authorization attached to organizational role rather than individual identity.

## 🧪 Positive and negative proof

The IT identity `sara.ibrahim` was used to demonstrate both sides of the authorization boundary:

```text
sara.ibrahim
     ↓
GG-IT
 ├── IT share        → ALLOWED ✅
 └── Finance share   → DENIED  🚫
```

The denied path is as important as the allowed path: it proves that group membership has operational authorization meaning and that the boundary is enforced rather than merely documented.

## ☁️ Future AWS authorization mapping

The original architecture also considered AWS job-function permission sets such as Cloud Admin, DevOps, Developer, Security and Auditor through IAM Identity Center.

Direct permission-set mapping was not implemented in this Free Plan lab because account/Organizations changes were intentionally not forced solely to satisfy the initial design.

When direct AWS-account SSO is introduced, the same authorization principle should be preserved:

```text
Employee
   ↓
Corporate group
   ↓
Central AWS workforce identity platform
   ↓
Job-function permission set
   ↓
Temporary AWS session
```

## 🔐 Security principle

Every production authorization role should have evidence for both:

- an intended action that succeeds,
- an out-of-scope action that is denied.

This converts least privilege from a policy statement into a testable control.

## ⚖️ Consequences

### Benefits

- Access follows organizational role.
- Onboarding and transfers become membership changes rather than per-resource edits.
- Offboarding is easier to reason about centrally.
- Positive and negative tests make authorization boundaries observable.
- The model maps naturally to future AWS permission sets.

### Trade-offs

- Group design must remain governed to avoid privilege accumulation.
- Nested or overlapping groups require careful review as the organization grows.
- Direct AWS permission-set enforcement remains a future extension of this Phase 04 lab.

## ✅ Result

The implemented Active Directory authorization model successfully demonstrated role-aligned group membership with both permitted and denied resource access.