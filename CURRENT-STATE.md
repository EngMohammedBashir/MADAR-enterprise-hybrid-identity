# Phase 04 — Current State

**Status:** AD CONNECTOR ACTIVE — HYBRID IDENTITY PATH PROVEN / AWS SERVICE CONSUMPTION NEXT  
**Date:** 2026-08-26  
**Primary theme:** Enterprise hybrid identity between on-premises Active Directory and AWS

## Executive checkpoint

Phase 04 now has a working routed hybrid path between the VMware corporate-identity lab and AWS, and the AWS Directory Service AD Connector for `madar.local` is **Active**.

This milestone was reached only after validating the path at packet level and fixing two separate failure layers:

```text
AD Connector attempt
        ↓
DNS unavailable / TCP 53
        ↓
Troubleshoot VPC routing + EC2 transit appliance
        ↓
Enable Linux IPv4 forwarding + transit NAT/forwarding
        ↓
Prove DNS request/reply across WireGuard
        ↓
Invalid credentials
        ↓
Reset/enable dedicated svc-adconnector account
        ↓
Fresh AD Connector attempt
        ↓
Stage = Active ✅
```

The next optional validation gate is to consume the directory from an AWS service such as Amazon WorkSpaces Personal and authenticate a synthetic domain user. During initial WorkSpaces registration discovery, the existing AD Connector appeared as an unregistered Directory Service directory but WorkSpaces reported its registration status as **Inoperable**. No WorkSpace has been launched and no additional WorkSpaces cost has been incurred yet. This is now a decision gate rather than a reason to disturb the proven network or Active Directory integration.

## Architecture implemented

```text
HOME / VMware                                      AWS / us-east-1

MADAR-DC01                                         MADAR-P04-VPC
192.168.14.10                                      10.50.0.0/16
AD DS + DNS                                             |
      |                                                |
      v                                                v
MADAR-WG01   ===== encrypted WireGuard =====      MADAR-P04-WG-HUB
192.168.14.30      10.200.0.2 <-> 10.200.0.1      EC2 network appliance
      |                                                |
      +---------------- routed path -------------------+
                                                       |
                                                       v
                                                  AD Connector
                                                  madar.local
                                                  Stage: ACTIVE
                                                       |
                                                       v
                                        AWS service consumption test
                                        (WorkSpaces candidate / decision gate)
```

## Network values used in the lab

| Component | Value |
|---|---|
| Corporate AD/DNS | `MADAR-DC01` — `192.168.14.10` |
| Local WireGuard router | `MADAR-WG01` — `192.168.14.30` |
| Local network | `192.168.14.0/24` |
| AWS VPC | `vpc-0371464657f10efb1` — `10.50.0.0/16` |
| Private subnet A | `subnet-0c6096cc338a611a1` |
| Private subnet B | `subnet-00661aa39bb01f61a` |
| AWS WG-HUB instance | `i-029deb16c4c36fd11` |
| AWS WG-HUB private IP | `10.50.1.132` |
| WireGuard transit | `10.200.0.0/30` |
| AWS tunnel IP | `10.200.0.1` |
| Local tunnel IP | `10.200.0.2` |
| WireGuard port | UDP `51820` |
| Active AD Connector | `d-90667da553` |

## What has been validated

### 1. Corporate identity baseline

- `MADAR-DC01` is the domain controller and DNS server for `madar.local`.
- Department OUs, synthetic workforce users and Global Security Groups exist.
- `MADAR-CLIENT01` joined the domain successfully.
- Domain-user login and GPO enforcement were validated.
- Positive and negative local authorization tests were captured.
- `sara.ibrahim` / `GG-IT` remains the preferred synthetic identity for a future end-to-end AWS authentication proof.

### 2. CGNAT-aware hybrid design

The Zain 5G lab connection sits behind carrier-grade NAT and does not provide a stable directly owned public IPv4 suitable for the classic static Customer Gateway model.

The lab therefore uses a self-managed routed WireGuard tunnel initiated outbound by `MADAR-WG01` and terminated on an EC2 network appliance. It is intentionally not described as AWS managed Site-to-Site VPN.

### 3. WireGuard tunnel

The tunnel is operational.

- Home peer: `10.200.0.2`.
- AWS peer: `10.200.0.1`.
- Handshake succeeds.
- Transfer counters increase in both directions.
- `PersistentKeepalive = 25` is used on the home side for CGNAT resilience.
- Bidirectional tunnel-side ping succeeds.
- AWS can ping `192.168.14.10` through the tunnel.

### 4. AD service reachability

From AWS, `MADAR-DC01` is reachable through the routed WireGuard path.

Validated services include:

- DNS resolution for `madar.local`,
- DNS TCP 53,
- Kerberos TCP 88,
- LDAP TCP 389,
- SMB TCP 445,
- password-change TCP 464,
- Global Catalog TCP 3268.

### 5. AWS private-subnet routing

Both AD Connector private subnets use the intended hybrid route path. The VPC route to `192.168.14.0/24` targets the EC2 WireGuard routing appliance.

### 6. EC2 routing appliance requirements

The EC2 instance is a transit router, not an ordinary endpoint. The working configuration requires:

- EC2 source/destination check disabled,
- Linux `net.ipv4.ip_forward = 1`,
- VPC route-table target to the appliance,
- WireGuard routes / AllowedIPs,
- Security Group allowance for VPC transit traffic,
- FORWARD rules between `10.50.0.0/16` and `192.168.14.0/24`,
- NAT/MASQUERADE for VPC-originated traffic entering `wg0`.

A key troubleshooting finding was that a healthy WireGuard handshake did **not** prove that VPC-originated packets could transit the appliance.

### 7. Packet-level proof

`tcpdump` captured Directory Service traffic arriving from AWS private addresses on `ens5`, being forwarded through `wg0`, receiving responses from `192.168.14.10`, and returning to the originating VPC address.

Observed DNS discovery included:

```text
A? madar.local
SRV? _ldap._tcp.madar.local
SRV? _kerberos._tcp.madar.local
A? madar-dc01.madar.local
```

The domain controller returned records pointing to `madar-dc01.madar.local` / `192.168.14.10`, including LDAP port 389 and Kerberos port 88.

This is strong evidence that AWS Directory Service performed real Active Directory discovery across the hybrid path.

### 8. Dedicated AD Connector identity

The dedicated service account is:

```text
svc-adconnector
```

During troubleshooting it was reset, enabled and unlocked. Validation showed:

```text
Enabled              : True
LockedOut            : False
PasswordExpired      : False
PasswordNeverExpires : True
```

No password is stored in this repository.

### 9. AD Connector ACTIVE

A fresh AD Connector creation completed successfully:

```text
Directory name : madar.local
Directory type : ADConnector
Directory ID   : d-90667da553
Stage          : Active
```

This closes the original Directory Service integration gate.

## Troubleshooting story

### Failure 1 — tunnel existed but end-to-end path initially failed

Symptoms included failed ping and no useful transit packets despite WireGuard configuration.

The investigation separated tunnel health from routed application connectivity. Bidirectional packet capture eventually proved public UDP/51820 exchange and a healthy WireGuard handshake.

### Failure 2 — AD Connector: DNS unavailable

Symptom:

```text
Connectivity issues detected: DNS unavailable (TCP port 53)
for IP: 192.168.14.10
```

The important discovery was that Directory Service traffic entered the EC2 appliance from the VPC, but the appliance initially was not forwarding it correctly.

Corrections included:

1. verify private-subnet routes,
2. disable EC2 source/destination check for the router,
3. enable Linux IPv4 forwarding,
4. allow VPC transit traffic in the hub Security Group,
5. configure FORWARD rules,
6. configure MASQUERADE for VPC-to-on-prem transit,
7. capture traffic simultaneously across `ens5` and `wg0`.

After the fix, TCP SYN/SYN-ACK and DNS query/reply traffic was visible in both directions.

### Failure 3 — AD Connector: invalid credentials

Once networking worked, the failure changed to:

```text
Configuration issues detected: Invalid credentials (bad username/password)
for IP: 192.168.14.10
```

This was useful evidence because the failure moved from Layer 3/4 connectivity to the authentication layer.

The `svc-adconnector` account was reset and validated, and a fresh connector attempt then reached `Active`.

### Important lesson

```text
WireGuard handshake ≠ routed application connectivity
Ping success ≠ AD service validation
TCP 53 success ≠ valid AD credentials
AD Connector Active = hybrid directory integration proven
```

Troubleshoot one layer at a time and preserve every working layer while moving upward.

## WorkSpaces discovery / current decision gate

After AD Connector became Active, Amazon WorkSpaces was evaluated as a possible **consumer** of the hybrid directory. The purpose is not to make WorkSpaces the project; it would only provide an end-to-end user authentication proof.

The WorkSpaces console currently shows the existing Directory Service object under **Unregistered directories**:

```text
Directory ID   : d-90667da553
Directory name : madar.local
Directory type : AD Connector
Registered     : False
Status         : Inoperable
```

No registration was completed and no WorkSpace was provisioned.

Before spending time or credits on this optional gate, determine why WorkSpaces labels the directory `Inoperable` and whether resolving that condition is low-risk. If it requires redesigning the already-proven hybrid path or introduces disproportionate cost, Phase 04 can close with AD Connector Active plus packet-level Active Directory discovery evidence, while documenting WorkSpaces as an optional future extension.

## IAM Identity Center decision

The earlier plan to continue directly from AD Connector into IAM Identity Center / permission sets was **not pursued** in this lab. Do not enable AWS Organizations or an IAM Identity Center organization instance merely to force that original architecture, especially where doing so changes account/credit behavior.

The repository should distinguish these two outcomes:

```text
PROVEN NOW
On-prem AD → WireGuard → AWS VPC → AD Connector ACTIVE

OPTIONAL NEXT PROOF
Domain user → AWS service consuming AD Connector (WorkSpaces candidate)
```

## Evidence to retain / capture

Existing hybrid evidence includes:

- `evidence/MADAR-WG01-Local-PreAWS-Validation.png`
- `evidence/AWS-VPC-Subnets-Validation.png`
- `evidence/AWS-Route-Tables-Validation.png`
- `evidence/phase04-wireguard-tunnel-evidence.png`
- `evidence/phase04-aws-to-onprem-ad-connectivity-evidence.png`

Additional evidence from this checkpoint should include:

- AD Connector `Active` status,
- packet capture showing AWS DNS/LDAP/Kerberos discovery and replies,
- dedicated service-account state without exposing credentials,
- WorkSpaces unregistered-directory screen if the optional path is documented.

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
Gate 7C  AD Connector Active                                       COMPLETE ✅
Gate 8   AWS service consumption / domain-user authentication       DECISION GATE
Gate 9   Security + validation evidence                            PENDING
Gate 10  VPN failure / recovery test                               PENDING
Gate 11  Documentation + architecture closeout                     PENDING
Gate 12  Cost/resource cleanup                                     PENDING
Gate 13  Phase 04 closeout                                         PENDING
```

## Immediate next decision

Do **not** redesign the tunnel, VPC, AD or AD Connector.

Choose one of two paths:

```text
Path A — stronger end-to-end proof
Investigate WorkSpaces "Inoperable"
        ↓
If low-risk, register madar.local
        ↓
Provision one minimal test WorkSpace
        ↓
Authenticate synthetic domain user (prefer sara.ibrahim)
        ↓
Capture proof
        ↓
Delete costly resource immediately

Path B — cost/time-aware closeout
Keep AD Connector ACTIVE proof
        ↓
Capture security/packet evidence
        ↓
Run VPN failure + recovery test
        ↓
Finish architecture + troubleshooting documentation
        ↓
Cost cleanup
        ↓
Phase 04 COMPLETE
```

The proven hybrid path must not be rebuilt unless a regression test demonstrates that it is actually broken.