# Initial Permission Matrix

This matrix is a design starting point. Exact AWS managed/custom policies will be selected only after tomorrow's service-by-service review.

| Role | Read | Operate workloads | Deploy/change app infra | Security visibility | IAM administration | Destructive admin |
|---|---:|---:|---:|---:|---:|---:|
| Cloud Admin | Yes | Yes | Yes | Yes | Controlled | Controlled |
| DevOps Engineer | Yes | Yes | Yes | Limited | No | Limited/No |
| Developer | Scoped | Scoped | Scoped | No | No | No |
| Security Team | Yes | No | No | Yes | Read only | No |
| Auditor | Yes | No | No | Read only | Read only | No |

## Test rule

Every role must have at least one **allowed** test and one **denied** test. A denied action is considered successful evidence when it matches the intended boundary.
