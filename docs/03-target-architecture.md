# Planned Target Identity Architecture

> Final integration details remain subject to execution-time validation against current AWS documentation and pricing.

```text
MADAR Corporate Workforce
          |
          v
Representative Microsoft Active Directory
(MADAR-DC01 on VMware)
          |
          | supported directory / identity integration
          v
AWS IAM Identity Center
          |
          +--> workforce groups
          +--> permission sets
          +--> MFA / SSO
          +--> AWS access portal
          +--> CLI SSO
          |
          v
Temporary AWS sessions
          |
          +--> Cloud Admin
          +--> DevOps
          +--> Developer
          +--> Security
          +--> Auditor
```

## Design rules

1. Do not create IAM users for ordinary workforce access unless a documented exception requires it.
2. Prefer group-to-permission assignment over per-user assignment.
3. Use temporary sessions rather than employee long-lived AWS access keys.
4. Require MFA where supported by the selected identity-source architecture.
5. Test denied actions intentionally.
6. Do not create a paid Directory Service component until the exact integration path and hourly cost are confirmed.
7. Keep the local AD VM powered off when not needed after the lab.

## Integration hold point

The corporate AD-to-AWS connection is the highest-risk design decision in this phase. It will not be improvised. Tomorrow's implementation will first verify the currently supported AWS path, prerequisites, network needs and cost before provisioning that portion.
