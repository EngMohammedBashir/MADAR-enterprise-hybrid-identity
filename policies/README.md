# Permission Policy Workspace

Custom policies belong here only when an AWS managed policy does not match the intended Phase 04 role boundary.

Rules:

- start from business actions, not service names,
- prefer least privilege,
- avoid wildcard administrative permissions unless the role explicitly requires them,
- document why a custom statement exists,
- pair every important permission with an access test,
- never store credentials or secrets in policy examples.
