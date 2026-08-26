# Phase 04 — Hybrid Identity Execution Record

This document began as the implementation plan for the AWS portion of Phase 04. It now records the **actual executed path**, including the architecture change from the original IAM Identity Center branch to the validated Amazon WorkSpaces hybrid identity proof.

## Final implemented architecture

```text
HOME / VMware                                 AWS / us-east-1

MADAR-DC01                                    MADAR-P04-VPC
192.168.14.10                                 10.50.0.0/16
AD DS + DNS                                        |
     |                                             |
     v                                             v
MADAR-WG01  ===== encrypted WireGuard =====   EC2 WG-HUB
192.168.14.30  10.200.0.2 <-> 10.200.0.1     Linux network appliance
                                                   |
                                                   v
                                             AD Connector
                                             d-90667da553
                                                   |
                                                   v
                                            WorkSpaces
                                       WSAMZN-I0F8R2FL
                                           10.50.13.89
                                                   |
                                                   v
                                           sara.ibrahim
```

## Naming / addressing record

Local:

- `MADAR-DC01` — domain controller / DNS, `192.168.14.10`
- `MADAR-CLIENT01` — Windows 11 domain client
- `MADAR-WG01` — WireGuard router, `192.168.14.30`

AWS:

- VPC: `MADAR-P04-VPC` — `10.50.0.0/16`
- WG-HUB EC2: `i-029deb16c4c36fd11`
- AD Connector: `d-90667da553`
- WorkSpaces subnet: `subnet-05e3c3e6fea490ac1` — `10.50.13.0/24` — `us-east-1c`
- WorkSpace: `ws-49q8s94dl`
- WorkSpace computer: `WSAMZN-I0F8R2FL`
- WorkSpace private IP: `10.50.13.89`

## Execution rules that remained important

1. Do not create directory integration before routed AD protocol tests pass.
2. Do not disable Windows Firewall globally as the normal solution.
3. Do not call the self-managed WireGuard design `AWS Site-to-Site VPN`.
4. Do not expose `MADAR-DC01` directly to the public Internet.
5. Treat the EC2 hub as a routing appliance; verify forwarding, SGs, routes and source/destination check.
6. Capture evidence at each gate before moving on.
7. Do not upgrade the AWS account merely to force an optional portfolio architecture branch.
8. Stop/delete paid temporary resources after validation.

## Stage 1 — Local directory baseline — COMPLETE

Validated:

- AD DS + DNS,
- `madar.local`,
- OUs and users,
- Global Security Groups,
- `MADAR-CLIENT01` domain join,
- Sara domain login,
- GPO / Domain firewall,
- IT share allowed,
- Finance share denied.

## Stage 2 — CGNAT-aware hybrid network — COMPLETE

The home lab is behind Zain 5G CGNAT. `MADAR-WG01` therefore initiates the tunnel outbound toward the AWS EC2 endpoint.

Required network-appliance properties were implemented:

- `net.ipv4.ip_forward = 1`,
- EC2 source/destination check disabled,
- VPC route to `192.168.14.0/24`,
- WireGuard AllowedIPs,
- Security Group allowance for VPC transit traffic,
- FORWARD / NAT rules,
- `PersistentKeepalive = 25` on the local side.

## Stage 3 — AD protocol validation — COMPLETE

The AWS side reached `MADAR-DC01` across the tunnel. Diagnostics included DNS, Kerberos, LDAP, SMB and related AD ports.

Packet capture was used to prove that Directory Service traffic crossed the EC2 appliance and received replies from the on-premises domain controller.

## Stage 4 — AD Connector — COMPLETE

The Connector troubleshooting sequence was:

```text
DNS unavailable
      ↓
fix VPC transit / forwarding path
      ↓
Invalid credentials
      ↓
validate/reset svc-adconnector
      ↓
AD Connector Active
```

Final Connector:

```text
Directory ID : d-90667da553
Domain       : madar.local
Stage        : Active
```

## Stage 5 — WorkSpaces network preparation — COMPLETE

WorkSpaces registration required compatible subnets in different supported AZs. A new private subnet was created:

```bash
AWS_PAGER="" aws ec2 create-subnet \
  --region us-east-1 \
  --vpc-id vpc-0371464657f10efb1 \
  --cidr-block 10.50.13.0/24 \
  --availability-zone us-east-1c \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=MADAR-P04-WorkSpaces-Private-C}]'
```

It was associated with the existing hybrid route table:

```bash
AWS_PAGER="" aws ec2 associate-route-table \
  --region us-east-1 \
  --route-table-id rtb-08c2d8c0ea2bac825 \
  --subnet-id subnet-05e3c3e6fea490ac1
```

This preserved the route:

```text
192.168.14.0/24 -> AWS WireGuard routing appliance
```

## Stage 6 — WorkSpaces directory registration — COMPLETE

The AD Connector was registered with Amazon WorkSpaces using supported subnets in `us-east-1b` and `us-east-1c`.

Final state:

```text
Registered : True
Status     : Active
Domain     : madar.local
```

## Stage 7 — Dedicated WorkSpaces OU — COMPLETE

Created:

```text
OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

The `svc-adconnector` service account was delegated the required computer-object permissions in that OU, and the WorkSpaces directory target OU was updated accordingly.

## Stage 8 — WorkSpace provisioning — COMPLETE

Selected:

```text
Bundle       : Standard Windows — Free tier eligible
Mode         : AutoStop after 1 hour
Root volume  : 80 GB
User volume  : 10 GB
User         : sara.ibrahim
```

Created:

```text
WorkSpace ID : ws-49q8s94dl
Computer     : WSAMZN-I0F8R2FL
Private IP   : 10.50.13.89
```

## Stage 9 — Domain join proof — COMPLETE

On the domain controller:

```powershell
Get-ADComputer -Filter * `
  -SearchBase "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" `
  -Properties DNSHostName,Enabled |
Select-Object Name,DNSHostName,Enabled,DistinguishedName |
Format-Table -AutoSize
```

Observed the AWS WorkSpace computer object in the on-premises AD.

## Stage 10 — Domain-user cloud authentication — COMPLETE

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

This proves the AWS-managed desktop consumed the on-premises corporate identity.

## Stage 11 — Healthy dependency baseline — COMPLETE

From the WorkSpace:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Observed TCP/53 success and DNS resolution from source `10.50.13.89`.

## Stage 12 — Controlled failure — COMPLETE

On `MADAR-WG01`:

```bash
sudo wg show
sudo wg-quick down wg0
```

The same WorkSpace tests then failed:

```text
TcpTestSucceeded : False
Resolve-DnsName  : timeout
```

## Stage 13 — Recovery — COMPLETE

Restored:

```bash
sudo wg-quick up wg0
sudo wg show
```

The WorkSpace again reached `192.168.14.10:53` and resolved `madar.local`.

## Original IAM Identity Center branch — DEFERRED

The initial execution plan included AWS Organizations, IAM Identity Center, permission sets, MFA, CLI SSO and CloudTrail workforce-session proof.

Those steps were **not executed** because the account remained intentionally on the AWS Free Plan and the project did not perform an account-plan upgrade or force Organizations changes solely to satisfy the initial architecture.

Future production extension:

```text
Corporate identity
      ↓
IAM Identity Center
      ↓
MFA / SSO
      ↓
Permission sets
      ↓
Temporary AWS sessions
```

## Evidence captured

- AD Connector Active
- WorkSpace computer object in on-prem AD
- WorkSpace domain-user authentication
- healthy WorkSpace-to-DC baseline
- controlled WireGuard failure
- successful recovery
- earlier WireGuard, routing, AD/GPO and local authorization evidence

## Closeout — IN PROGRESS

After documentation review:

```text
Stop/delete WorkSpace
      ↓
Deregister WorkSpaces directory
      ↓
Delete AD Connector if not reused
      ↓
Terminate temporary WG-HUB EC2
      ↓
Release public address if unused
      ↓
Remove temporary Phase 04 network artifacts as appropriate
      ↓
Power off local VMs
      ↓
Bills / Credits / residual-resource audit
      ↓
PHASE 04 ACCEPTED
```
