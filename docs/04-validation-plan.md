# Phase 04 Validation Plan

## Validation philosophy

A green console status alone is not acceptance. Each important layer must be proven with observed behavior.

## Completed authentication tests

- [x] `sara.ibrahim` exists in the corporate `madar.local` Active Directory.
- [x] AD Connector reached `Active`.
- [x] Amazon WorkSpaces successfully discovered the domain user.
- [x] A WorkSpace was provisioned for `sara.ibrahim`.
- [x] The WorkSpace computer object was created in the on-premises WorkSpaces OU.
- [x] Sara authenticated successfully to the WorkSpace using her domain identity.

Inside the WorkSpace:

```powershell
whoami
hostname
$env:USERDNSDOMAIN
```

Observed:

```text
madar\sara.ibrahim
WSAMZN-I0F8R2FL
MADAR.LOCAL
```

## Completed connectivity tests

### Healthy baseline

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Expected and observed:

```text
SourceAddress    : 10.50.13.89
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

### Controlled failure

The WireGuard interface on `MADAR-WG01` was intentionally stopped:

```bash
sudo wg-quick down wg0
```

The same WorkSpace tests then showed:

```text
TcpTestSucceeded : False
Resolve-DnsName  : timeout
```

### Recovery

The tunnel was restored:

```bash
sudo wg-quick up wg0
```

The WorkSpace-to-DC tests returned to success.

This proves the expected dependency:

```text
WorkSpace
   ↓
AD Connector / VPC routing
   ↓
WireGuard
   ↓
On-prem AD/DNS
```

## Local authorization tests already completed

Before AWS integration, Sara's group membership was used for local positive/negative authorization proof:

```text
Sara / GG-IT
   ├── IT share       -> ALLOWED
   └── Finance share  -> DENIED
```

The intentional denial is successful evidence of an authorization boundary.

## IAM Identity Center / SSO tests

These were part of the original target but were not executed in this Free Plan lab because the project did not upgrade the account or force AWS Organizations changes only to satisfy the initial design.

Therefore the repository must not mark the following as completed:

- IAM Identity Center account SSO,
- MFA through Identity Center,
- permission-set authorization,
- CLI SSO,
- CloudTrail SSO principal validation.

They remain future production-extension tests.

## Acceptance rule for the implemented lab

Phase 04 technical validation is accepted when all of the following are true:

```text
Corporate AD source works
        +
Hybrid routed network works
        +
AD Connector Active
        +
AWS-managed WorkSpace joins madar.local
        +
Existing domain user authenticates
        +
Failure causes expected connectivity loss
        +
Recovery restores the path
        +
Cost/resource cleanup is completed
```
