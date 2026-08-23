# Positive Access Tests

Each test must record:

```text
Identity
Group
Permission set
Action
Expected result
Observed result
Evidence
```

Planned examples:

- Cloud Admin performs approved administrative action.
- DevOps Engineer performs approved infrastructure/deployment action.
- Developer performs scoped workload action.
- Security Team reads security/audit data.
- Auditor reads allowed inventory/evidence.

A successful login alone does not satisfy these tests; an authorized AWS action must be observed.
