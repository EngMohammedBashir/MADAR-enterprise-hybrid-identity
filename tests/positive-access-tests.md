# Positive Access Tests

## Implemented positive tests

### Local authorization

```text
Identity : sara.ibrahim
Group    : GG-IT
Action   : Access IT departmental share
Expected : Allowed
Observed : Allowed
Evidence : evidence/Sara-IT-Share-Access-Success.png
```

### AWS-managed cloud desktop authentication

```text
Identity : madar\sara.ibrahim
Service  : Amazon WorkSpaces Personal
Computer : WSAMZN-I0F8R2FL
Expected : Corporate domain authentication succeeds
Observed : Success
Evidence : evidence/workspaces-hybrid-ad-authentication-validation.png
```

### Hybrid WorkSpace-to-AD connectivity

```text
Source   : 10.50.13.89
Target   : 192.168.14.10:53
Expected : TCP succeeds and madar.local resolves
Observed : Success
Evidence : evidence/workspaces-to-onprem-baseline-connectivity.png
```

### Recovery

After an intentional WireGuard outage, the tunnel was restored and the same TCP/DNS tests succeeded again.

Evidence:

`evidence/workspaces-vpn-failure-recovery-validation.png`

## Future direct AWS-account authorization tests

Cloud Admin / DevOps / Developer / Security / Auditor permission-set tests remain future production-extension work because IAM Identity Center was not implemented under the Free Plan account guardrail.
