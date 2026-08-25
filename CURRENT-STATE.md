# Phase 04 — Current State

**Status:** ACTIVE — HYBRID NETWORK PATH VALIDATED / AD CONNECTOR CREDENTIAL FIX NEXT  
**Date:** 2026-08-25  
**Primary theme:** Enterprise workforce identity and AWS access governance

## Executive checkpoint

Phase 04 now has a working routed hybrid path between the VMware corporate-identity lab and AWS.

The important milestone is not merely that WireGuard shows a handshake. AWS-originated traffic was traced across the EC2 routing appliance, through the encrypted tunnel, into the on-premises network, and to `MADAR-DC01` at `192.168.14.10`. DNS TCP/53 request/response traffic was observed successfully.

The latest AD Connector attempt therefore progressed from a **network failure** to an **authentication failure**:

```text
Earlier failure: DNS unavailable / TCP 53
        ↓
Routing + EC2 appliance troubleshooting
        ↓
AWS-to-on-prem DNS traffic proven
        ↓
Latest failure: Invalid credentials
```

This is meaningful progress: the network path is now proven. The next action is to validate/reset the dedicated `svc-adconnector` credential and retry the connector without changing the working network.

## Architecture now implemented

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
      |                                                +-- private subnet routing
      |                                                |
      +---------------- routed path -------------------+
                                                       |
                                                       v
                                                  AD Connector
                                                       |
                                                       v
                                             IAM Identity Center
```

### Network values used in the lab

| Component | Value |
|---|---|
| Corporate AD/DNS | `MADAR-DC01` — `192.168.14.10` |
| Local WireGuard router | `MADAR-WG01` — `192.168.14.30` |
| Local network | `192.168.14.0/24` |
| AWS VPC | `10.50.0.0/16` |
| WireGuard transit | `10.200.0.0/30` |
| AWS tunnel IP | `10.200.0.1` |
| Local tunnel IP | `10.200.0.2` |
| WireGuard port | UDP `51820` |

## What has been validated

### 1. Corporate identity baseline

- `MADAR-DC01` is the domain controller for `madar.local`.
- Department OUs, synthetic workforce users and Global Security Groups exist.
- `MADAR-CLIENT01` joined the domain successfully.
- Domain-user login and GPO enforcement were validated.
- Positive and negative local authorization tests were captured.

### 2. CGNAT-aware design

The Zain 5G lab connection sits behind carrier-grade NAT and does not provide a stable, directly owned public IPv4 suitable for the classic static Customer Gateway model.

The selected lab solution is therefore a **self-managed routed WireGuard tunnel** initiated outbound by `MADAR-WG01` and terminated on an EC2 network appliance. It is intentionally not described as AWS managed Site-to-Site VPN.

### 3. WireGuard tunnel

The tunnel is operational.

- `MADAR-WG01` peer: `10.200.0.2/30`.
- AWS `WG-HUB` peer: `10.200.0.1/30`.
- Handshake succeeds.
- Transfer counters increase in both directions.
- `PersistentKeepalive = 25` is used on the home side for CGNAT resilience.
- Tunnel-side ping to `10.200.0.1` succeeds.

### 4. Local AD reachability through the tunnel

From the AWS side, `MADAR-DC01` became reachable through the routed WireGuard path.

Validated services include:

- DNS resolution for `madar.local`,
- DNS TCP 53,
- Kerberos TCP 88,
- LDAP TCP 389,
- SMB TCP 445,
- additional AD service checks including TCP 135, 464 and 3268.

### 5. AWS private-subnet routing

Both AD Connector private subnets are associated with the intended private route table.

The route table contains:

```text
10.50.0.0/16       -> local
192.168.14.0/24    -> MADAR-P04-WG-HUB EC2 appliance
```

The route to the on-premises CIDR is `active`.

### 6. EC2 routing appliance behavior

The AWS WireGuard instance is acting as a router, not as an ordinary destination host.

The working path requires:

- Linux IPv4 forwarding,
- WireGuard routes/AllowedIPs,
- VPC route-table target to the appliance,
- EC2 source/destination-check handling appropriate for a routing appliance,
- Security Group allowance for VPC transit traffic,
- forwarding/NAT rules for the routed networks.

The AWS hub Security Group originally allowed only UDP/51820. VPC transit traffic from `10.50.0.0/16` therefore could not reach the appliance. Adding the required VPC-side allowance removed that boundary.

### 7. Packet-level proof

Troubleshooting used `tcpdump` rather than assuming that a successful tunnel handshake meant end-to-end application connectivity.

The decisive proof was observing AWS-originated TCP/53 traffic reach `192.168.14.10` and receive replies from the domain controller. A complete TCP handshake was visible.

That changed the diagnosis from:

```text
"the directory cannot reach DNS"
```

to:

```text
"the directory reaches the domain controller; authentication is now the failing layer"
```

## Troubleshooting story worth retaining

This phase intentionally keeps the failures because they demonstrate a repeatable troubleshooting method.

### Failure 1 — AD Connector: DNS unavailable

Symptom:

```text
Connectivity issues detected: DNS unavailable (TCP port 53)
for IP: 192.168.14.10
```

Investigation:

1. verified private-subnet route-table associations,
2. verified the `192.168.14.0/24` route targeted `WG-HUB`,
3. inspected Directory Service ENIs,
4. compared packet capture on `ens5` and `wg0`,
5. inspected the AWS-created Directory Service Security Group,
6. inspected the `WG-HUB` Security Group,
7. found that the hub accepted WireGuard UDP/51820 but did not accept VPC transit traffic,
8. added the required VPC-side Security Group allowance,
9. retained the Linux forwarding/NAT path,
10. repeated the connector test and captured DNS request/reply traffic.

Result: **network/DNS connectivity moved from failed to proven.**

### Failure 2 — AD Connector: invalid credentials

Latest symptom:

```text
Configuration issues detected: Invalid credentials (bad username/password)
for IP: 192.168.14.10
```

Interpretation: this is a later-layer failure and confirms that the connector reached the AD/DNS environment far enough to attempt authentication.

Next action: validate the dedicated `svc-adconnector` account state and password on `MADAR-DC01`, test the credential independently, then create a fresh connector attempt.

## Dedicated connector identity

A dedicated Active Directory service account was created:

```text
svc-adconnector
```

The account was enabled and configured for the lab integration. Passwords are never stored in this repository, screenshots, shell history or documentation.

## Evidence captured

New hybrid evidence currently committed includes:

- `evidence/MADAR-WG01-Local-PreAWS-Validation.png`
- `evidence/AWS-VPC-Subnets-Validation.png`
- `evidence/AWS-Route-Tables-Validation.png`
- `evidence/phase04-wireguard-tunnel-evidence.png`
- `evidence/phase04-aws-to-onprem-ad-connectivity-evidence.png`

These complement the existing Active Directory, domain-join, GPO and local least-privilege evidence.

## Cost-aware pause state

At the end of the session:

- the failed AD Connector was deleted,
- `MADAR-P04-WG-HUB` EC2 was stopped rather than terminated so the lab can resume,
- local VMware VMs can remain powered off until needed,
- persistent low-cost storage/public-address considerations remain part of the final cleanup audit.

No working routing configuration should be rebuilt tomorrow. Resume from the credential layer.

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
Gate 7B  AD Connector authentication                               BLOCKED — CREDENTIALS
Gate 7C  AD Connector Active                                       PENDING
Gate 8   IAM Identity Center                                       PENDING
Gate 9   Permission sets / account assignments                     PENDING
Gate 10  SSO + MFA + temporary console/CLI sessions                PENDING
Gate 11  Positive + negative least-privilege tests                 PENDING
Gate 12  Joiner / mover / leaver lifecycle                         PENDING
Gate 13  CloudTrail audit evidence                                 PENDING
Gate 14  Cost/resource cleanup + closeout                          PENDING
```

## Resume point

Next session should start in this order:

```text
Power on MADAR-DC01
        ↓
Power on MADAR-WG01
        ↓
Start MADAR-P04-WG-HUB
        ↓
Verify WireGuard handshake
        ↓
Verify DNS path still works
        ↓
Validate/reset svc-adconnector credential
        ↓
Test credential independently
        ↓
Create AD Connector
        ↓
Target: Stage = Active
```

Do **not** redesign the tunnel, VPC or route tables unless a regression test proves that layer has broken.