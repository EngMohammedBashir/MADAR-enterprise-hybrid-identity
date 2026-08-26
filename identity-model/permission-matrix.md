# Future AWS Permission Matrix

This matrix is retained as the **future production design** for direct AWS-account workforce access through IAM Identity Center.

It was not implemented in the current Free Plan lab and should not be read as validated evidence.

| Role | Read | Operate workloads | Deploy/change app infra | Security visibility | IAM administration | Destructive admin |
|---|---:|---:|---:|---:|---:|---:|
| Cloud Admin | Yes | Yes | Yes | Yes | Controlled | Controlled |
| DevOps Engineer | Yes | Yes | Yes | Limited | No | Limited/No |
| Developer | Scoped | Scoped | Scoped | No | No | No |
| Security Team | Yes | No | No | Yes | Read only | No |
| Auditor | Yes | No | No | Read only | Read only | No |

## Current implemented authorization proof

Phase 04 currently proves group-based authorization inside the corporate AD environment:

```text
sara.ibrahim / GG-IT
   ├── IT share       -> ALLOWED
   └── Finance share  -> DENIED
```

It also proves centralized corporate authentication to Amazon WorkSpaces through AD Connector.

## Future test rule

When IAM Identity Center is intentionally deployed in a production AWS Organization, every role above must have at least one **allowed** action and one **denied** action. A denied action counts as successful evidence when it matches the intended privilege boundary.
