# Runbook — Offboard Workforce User

1. Disable/remove the employee identity according to the corporate-directory policy.
2. Remove workforce group memberships if required by the identity-source model.
3. Confirm the user cannot establish a new AWS SSO session.
4. Verify direct employee IAM users/access keys were not created for normal workforce access.
5. Review recent audit activity.
6. Record revocation evidence and timing.

The goal is centralized revocation: changing the corporate identity should remove future AWS access without hunting for independent credentials in multiple services.
