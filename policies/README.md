# Permission Policy Workspace

This folder is reserved for future direct AWS-account workforce authorization policies.

Phase 04's implemented lab outcome validated **hybrid corporate identity through Amazon WorkSpaces**, not IAM Identity Center permission sets. Therefore no fake or placeholder AWS permission policy is committed merely to make the repository look more complete.

## Current rule

```text
Implemented now
Corporate AD -> WireGuard -> AD Connector -> WorkSpaces -> domain-user authentication

Future production extension
Corporate identity -> IAM Identity Center -> group-based permission sets -> temporary AWS sessions
```

When direct AWS-account authorization is implemented in a later environment:

- start from business actions, not service names,
- prefer least privilege,
- avoid wildcard administrative permissions unless a role explicitly requires them,
- document why every custom statement exists,
- pair every important permission with an allowed and denied test,
- never store credentials, secrets or session tokens in policy examples.
