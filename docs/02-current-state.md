# Current-State Identity Assessment

## Scenario

MADAR already has corporate employee identities outside AWS. The Phase 04 lab will represent that system with a small VMware-hosted Windows Server / Active Directory environment.

## Current risk

Without centralized workforce identity, AWS access would tend toward:

```text
employee
   ↓
IAM user
   ↓
password / long-lived access key
   ↓
manual permission changes
```

Problems include credential sprawl, difficult offboarding, inconsistent least privilege, weak group governance and poor audit clarity.

## Desired transition

```text
Corporate identity
      ↓
Central SSO
      ↓
Group-based permission sets
      ↓
Temporary AWS session
      ↓
Auditable access
```

## Scope boundary

This phase focuses on human workforce access. Application/service identities, workload IAM roles and broad multi-account governance belong to their own phases unless Phase 04 needs them for a specific test.
