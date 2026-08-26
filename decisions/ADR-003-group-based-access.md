# ADR-003 — Group-Based Authorization

**Status:** Accepted; local authorization implemented / direct AWS permission-set mapping deferred

## Decision

Use groups as the primary authorization model rather than assigning ordinary access directly to individual users.

## Implemented proof

The source-side Active Directory model uses departmental Global Security Groups:

- `GG-Management`
- `GG-IT`
- `GG-Finance`
- `GG-HR`
- `GG-Sales`

The IT identity `sara.ibrahim` was used for a positive and negative authorization test:

```text
Sara / GG-IT
   ├── IT share       -> ALLOWED
   └── Finance share  -> DENIED
```

This proves that group membership has operational authorization meaning rather than being directory decoration.

## Direct AWS authorization branch

The original design planned AWS permission sets such as Cloud Admin, DevOps, Developer, Security and Auditor through IAM Identity Center.

That branch was not implemented in the Free Plan lab because the account was not upgraded and AWS Organizations changes were not forced solely to satisfy the initial architecture.

## Future production rule

For direct AWS-account access, preserve the same principle:

```text
Employee
   ↓
Corporate group
   ↓
Central AWS workforce identity platform
   ↓
Job-function permission set
   ↓
Temporary session
```

Every production role should demonstrate an intended allowed action and at least one denied action that proves the privilege boundary.
