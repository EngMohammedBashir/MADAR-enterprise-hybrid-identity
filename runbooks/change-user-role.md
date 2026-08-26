# Runbook — Change Workforce Role

## Implemented directory model

1. Identify the employee's current and target department/team.
2. Change corporate AD group membership rather than granting direct per-user access.
3. Verify the user's source identity remains in the correct OU.
4. Re-test any local authorization boundary affected by the group change.
5. If the employee uses Amazon WorkSpaces, keep the WorkSpace computer object in the dedicated `WorkSpaces` OU; the user role change should not require moving the computer object.
6. Sign out/re-authenticate where required to refresh effective group membership.
7. Verify the old permission path is no longer available.
8. Verify the new intended path works.
9. Record evidence.

## Important distinction

```text
User group membership
  = authorization / job-function change

WorkSpace computer OU
  = device organization / domain-join placement
```

Changing Sara from one business role to another should modify identity/group authorization, not rebuild the hybrid network or AD Connector.

## Future production extension

When IAM Identity Center is intentionally deployed in a production AWS Organization, the mover workflow should additionally verify that old AWS permission-set access disappears and the new temporary-session permissions become effective.
