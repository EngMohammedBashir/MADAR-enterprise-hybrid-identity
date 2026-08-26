# Phase 04 — Current State

**Status:** HYBRID IDENTITY VALIDATED — WORKSPACES END-TO-END PROOF COMPLETE / CLEANUP PENDING  
**Date:** 2026-08-26  
**Primary theme:** Enterprise hybrid identity between on-premises Active Directory and AWS

## Executive checkpoint

Phase 04 now has a fully demonstrated hybrid identity path from a VMware-hosted corporate Active Directory to an AWS-managed end-user desktop.

The final validated path is:

```text
sara.ibrahim
      ↓
Amazon WorkSpaces Personal
WSAMZN-I0F8R2FL / 10.50.13.89
      ↓
AWS Directory Service AD Connector
      ↓
AWS VPC private routing
      ↓
EC2 WireGuard routing appliance
      ↓
WireGuard tunnel
      ↓
MADAR-WG01
      ↓
MADAR-DC01 / 192.168.14.10
      ↓
madar.local
```

The project did not stop at `AD Connector = Active`. It proved that an AWS WorkSpace was created for `sara.ibrahim`, that its computer account appeared inside the on-premises AD, that Sara authenticated successfully to the cloud desktop with her corporate domain identity, and that WorkSpace-to-domain-controller connectivity failed and recovered exactly as expected when the WireGuard tunnel was intentionally stopped and restarted.

## Key validated resources

| Component | Value / state |
|---|---|
| Corporate AD/DNS | `MADAR-DC01` — `192.168.14.10` |
| Domain | `madar.local` |
| Local WireGuard router | `MADAR-WG01` — `192.168.14.30` |
| AWS VPC | `vpc-0371464657f10efb1` — `10.50.0.0/16` |
| Existing private subnet A | `subnet-0c6096cc338a611a1` — `10.50.11.0/24` — `us-east-1a` |
| Existing private subnet B | `subnet-00661aa39bb01f61a` — `10.50.12.0/24` — `us-east-1b` |
| Added WorkSpaces subnet | `subnet-05e3c3e6fea490ac1` — `10.50.13.0/24` — `us-east-1c` |
| Hybrid route table | `rtb-08c2d8c0ea2bac825` |
| AWS WG-HUB instance | `i-029deb16c4c36fd11` |
| WireGuard transit | `10.200.0.0/30` |
| AD Connector | `d-90667da553` — Active |
| WorkSpaces target OU | `OU=WorkSpaces,OU=MADAR,DC=madar,DC=local` |
| WorkSpace | `ws-49q8s94dl` |
| WorkSpace computer | `WSAMZN-I0F8R2FL` |
| WorkSpace private IP | `10.50.13.89` |
| WorkSpace user | `sara.ibrahim` |

## What happened after AD Connector became Active

### 1. WorkSpaces directory registration

The existing AD Connector was registered successfully with Amazon WorkSpaces.

Observed state:

```text
Directory ID : d-90667da553
Directory    : madar.local
Registered   : True
Status       : Active
```

WorkSpaces needed supported subnets in different Availability Zones. `us-east-1a` was not offered by the WorkSpaces registration UI for the second selection, so a new private subnet was created in `us-east-1c`:

```text
subnet-05e3c3e6fea490ac1
CIDR  : 10.50.13.0/24
AZ    : us-east-1c
AZ ID : use1-az4
```

It was associated with the existing hybrid route table so `192.168.14.0/24` continued to route through the WireGuard EC2 appliance.

### 2. Dedicated WorkSpaces OU

A dedicated OU was created:

```text
OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

The `svc-adconnector` account was delegated the computer-object permissions required for WorkSpaces domain join.

The WorkSpaces directory was then configured to use that OU as its target domain/organizational unit.

### 3. WorkSpace provisioning

One test desktop was provisioned with cost guardrails:

```text
Bundle       : Standard with Windows 10 (Server 2022 based)
Eligibility  : Free tier eligible
Running mode : AutoStop after 1 hour
Root volume  : 80 GB
User volume  : 10 GB
Assigned user: sara.ibrahim
```

Result:

```text
WorkSpace ID : ws-49q8s94dl
Computer     : WSAMZN-I0F8R2FL
Private IP   : 10.50.13.89
```

### 4. On-premises domain join proof

On `MADAR-DC01`:

```powershell
Get-ADComputer -Filter * `
  -SearchBase "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" `
  -Properties DNSHostName,Enabled |
Select-Object Name,DNSHostName,Enabled,DistinguishedName |
Format-Table -AutoSize
```

Observed:

```text
Name            : WSAMZN-I0F8R2FL
DNSHostName     : WSAMZN-I0F8R2FL.madar.local
Enabled         : True
DN              : CN=WSAMZN-I0F8R2FL,OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

### 5. Domain-user WorkSpaces login proof

Inside the WorkSpace:

```powershell
whoami
hostname
$env:USERDNSDOMAIN
ipconfig | findstr /i "IPv4 DNS"
```

Observed:

```text
madar\sara.ibrahim
WSAMZN-I0F8R2FL
MADAR.LOCAL
IPv4 Address : 10.50.13.89
```

This is the key end-to-end identity proof: the identity exists in the on-premises AD, while the desktop is managed by AWS.

## Failure / recovery validation

### Healthy baseline

From the WorkSpace:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Observed:

```text
SourceAddress    : 10.50.13.89
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

### Controlled VPN failure

On `MADAR-WG01`:

```bash
sudo wg show
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
sudo wg show
```

The WorkSpace returned to:

```text
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

Operational story:

```text
Healthy
   ↓
VPN intentionally disabled
   ↓
WorkSpace loses on-prem AD/DNS reachability
   ↓
VPN restored
   ↓
Connectivity recovers
```

## IAM Identity Center decision

The original target architecture assumed IAM Identity Center, SSO/MFA and permission sets. That branch was not forced in this lab because the account was intentionally kept on the AWS Free Plan and the project guardrail was **no account-plan upgrade and no Organizations changes solely to satisfy the initial design**.

The validated Phase 04 outcome is therefore:

```text
PROVEN
On-prem AD
   ↓
WireGuard hybrid network
   ↓
AD Connector Active
   ↓
Amazon WorkSpaces
   ↓
Domain-joined cloud desktop
   ↓
Corporate domain-user login
   ↓
Failure/recovery validation
```

IAM Identity Center remains a future production extension when its organizational prerequisites are intentionally available.

## Cost checkpoint

The account remained on the AWS Free Plan.

At the WorkSpaces validation checkpoint:

```text
Credits issued              : $180.00
Estimated used              : $3.53
Estimated remaining         : $176.47
Amazon WorkSpaces           : applicable credit product
Selected Standard bundle    : Free tier eligible
```

One WorkSpace was used with AutoStop. No NAT Gateway was created for the WorkSpaces test.

## Evidence captured

- `evidence/ad-connector-active-success.png`
- `evidence/workspaces-onprem-ad-domain-join-verified.png`
- `evidence/workspaces-hybrid-ad-authentication-validation.png`
- `evidence/workspaces-to-onprem-baseline-connectivity.png`
- `evidence/workspaces-to-onprem-vpn-failure-test.png`
- `evidence/workspaces-vpn-failure-recovery-validation.png`
- existing local AD, GPO, authorization, WireGuard and AWS routing evidence

## Current execution gates

```text
Gate 1   Business problem / current-state identity assessment       COMPLETE
Gate 2   VMware Windows Server + AD DS lab baseline                COMPLETE
Gate 3   Workforce users/groups + local authorization validation   COMPLETE
Gate 4   CGNAT-aware hybrid connectivity architecture              COMPLETE
Gate 5A  Local MADAR-WG01 readiness                                COMPLETE
Gate 5B  AWS VPC + EC2 WG-HUB + WireGuard handshake                COMPLETE
Gate 6   AWS -> on-prem routed AD/DNS connectivity                 COMPLETE
Gate 7A  AD Connector network path                                 COMPLETE
Gate 7B  AD Connector authentication                               COMPLETE
Gate 7C  AD Connector Active                                       COMPLETE
Gate 8   AWS service consumption / domain-user authentication       COMPLETE ✅
Gate 9   WorkSpace domain join / target OU                         COMPLETE ✅
Gate 10  VPN failure / recovery test                               COMPLETE ✅
Gate 11  Documentation + architecture closeout                     IN PROGRESS
Gate 12  Cost/resource cleanup                                     PENDING
Gate 13  Phase 04 closeout                                         PENDING
```

## Immediate next actions

Do not build more infrastructure merely to add services.

```text
1. Finish repository documentation
2. Update master MADAR transformation repository
3. Stop temporary compute while documentation is reviewed
4. Perform final resource cleanup
5. Review Bills / credits / residual resources
6. Mark Phase 04 accepted
```
