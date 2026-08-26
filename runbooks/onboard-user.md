# Runbook — Onboard Workforce User

## Implemented Phase 04 path

1. Create or enable the employee in the corporate `madar.local` directory.
2. Place the employee in the correct departmental security group.
3. Verify the identity can authenticate locally before AWS integration.
4. Confirm the AD Connector is `Active`.
5. If a WorkSpace is required, assign the employee to the registered `madar.local` WorkSpaces directory.
6. Use the dedicated WorkSpaces computer OU; do not move the user object into the computer OU.
7. Provision the WorkSpace with the approved cost profile (`AutoStop` for the lab).
8. Verify the WorkSpace computer object is created in `OU=WorkSpaces,OU=MADAR,DC=madar,DC=local`.
9. Verify the employee can sign in using the existing corporate AD password.
10. Verify `whoami`, hostname and domain values inside the WorkSpace.
11. Record evidence and stop the WorkSpace when testing is complete.

## Example validation

```powershell
whoami
hostname
$env:USERDNSDOMAIN
```

For the tested IT user, the observed identity was:

```text
madar\sara.ibrahim
```

## Future production extension

If IAM Identity Center is intentionally available in a future AWS Organization, onboarding can additionally include group-to-permission-set assignment, MFA and temporary AWS-account sessions.

Do not create an employee-specific long-lived AWS access key as part of normal onboarding.
