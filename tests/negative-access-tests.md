# Negative Access Tests

Denied behavior is first-class evidence when it matches the intended boundary.

## Implemented negative tests

### Local authorization denial

```text
Identity : sara.ibrahim
Group    : GG-IT
Action   : Access Finance departmental share
Expected : Denied
Observed : Denied
Evidence : evidence/Sara-Finance-Access-Denied.png
```

This is a successful least-privilege result.

### Hybrid network dependency failure

The WireGuard tunnel was intentionally stopped:

```bash
sudo wg-quick down wg0
```

From the AWS WorkSpace:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Observed:

```text
TcpTestSucceeded : False
DNS query        : timeout
```

Evidence:

`evidence/workspaces-to-onprem-vpn-failure-test.png`

This is not an authorization denial; it is an intentional negative availability/dependency test proving that the WorkSpace really depends on the designed hybrid path.

## Future direct AWS-account negative tests

The original Developer/Auditor/Security permission-set denial tests remain future production-extension work because IAM Identity Center was not implemented under the Free Plan account guardrail.
