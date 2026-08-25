# Phase 04 — Current State

**Status:** ACTIVE — LOCAL IDENTITY + LOCAL WIREGUARD GATEWAY VALIDATED / AWS HUB NEXT  
**Date:** 2026-08-25  
**Primary theme:** Enterprise workforce identity and AWS access governance

## Starting point

Phase 03 is complete and cleaned up. Phase 04 has a working local identity source and a validated local WireGuard routing gateway. The next execution step is the AWS VPC + EC2 WireGuard hub.

## Completed local identity baseline

```text
MADAR-DC01
├── Windows Server 2025
├── static IPv4: 192.168.14.10
├── AD DS + DNS
├── domain: madar.local
├── department OUs
├── Global Security Groups
└── GPO-IT-Security

MADAR-CLIENT01
├── Windows 11 Pro
├── joined to madar.local
├── domain authentication validated
└── GPO / firewall enforcement validated
```

Synthetic workforce identities:

| Department | User | Group |
|---|---|---|
| Management | Ahmed Ali (`ahmed.ali`) | `GG-Management` |
| IT | Sara Ibrahim (`sara.ibrahim`) | `GG-IT` |
| Finance | Omar Hassan (`omar.hassan`) | `GG-Finance` |
| HR | Noura Saleh (`noura.saleh`) | `GG-HR` |
| Sales | Khalid Mansour (`khalid.mansour`) | `GG-Sales` |

## Local validation completed

- domain-controller role and `madar.local` membership verified,
- users distributed into departmental OUs,
- group membership verified using PowerShell,
- Windows 11 client joined to the domain,
- domain-user login validated,
- `GPO-IT-Security` confirmed as applied,
- Domain firewall profile confirmed enabled,
- IT share access succeeded for the IT user,
- Finance share access was denied for the same IT user as an intentional authorization boundary.

## Hybrid-connectivity preflight result

The local Internet connection is Zain 5G. Testing established:

- router WAN IPv4 is private (`10.x.x.x`),
- a carrier NAT exists between the router and the public Internet,
- the observed public IPv4 changed after router restart/reconnect,
- the lab therefore does not have a stable, directly owned public IPv4 suitable for the classic static-address Customer Gateway design.

The classic AWS managed Site-to-Site VPN path was therefore not selected for this home-lab implementation.

## Selected lab connectivity architecture

Phase 04 uses a **self-managed routed WireGuard tunnel**:

```text
MADAR-DC01 (192.168.14.10)
        |
        v
MADAR-WG01 (192.168.14.30)
        |
        | outbound-initiated WireGuard
        v
EC2 WG-HUB (AWS network appliance)
        |
        v
Phase 04 VPC
        |
        v
AD Connector
        |
        v
IAM Identity Center
```

The home side initiates the tunnel outbound, so the local public IPv4 does not need to be a stable inbound endpoint. The EC2 instance acts as the AWS-side routed network appliance.

This is intentionally documented as **self-managed WireGuard on an EC2 network appliance**, not as AWS managed Site-to-Site VPN.

Decision record: `decisions/ADR-004-cgnat-wireguard-hybrid-connectivity.md`.

## MADAR-WG01 — completed local gateway baseline

The retired Phase 03 Ubuntu VM was intentionally repurposed as the local routing gateway instead of creating another VM on the RAM-constrained workstation.

```text
MADAR-WG01
├── Ubuntu 24.04
├── hostname: madar-wg01
├── static IPv4: 192.168.14.30/24
├── default gateway: 192.168.14.2
├── SSH validated on the static address
├── WireGuard installed
├── IPv4 forwarding = 1 and persistent
├── WireGuard keypair created
├── private key permissions = 600
└── UFW left inactive rather than introducing a new firewall layer mid-lab
```

Validated from `MADAR-WG01` to `MADAR-DC01`:

- ICMP reachability to `192.168.14.10`,
- `madar.local` DNS resolution through `192.168.14.10`,
- TCP 53 DNS,
- TCP 88 Kerberos,
- TCP 389 LDAP,
- TCP 445 SMB.

The local gateway is therefore ready for its AWS peer configuration. The final `wg0` peer block is intentionally deferred until the AWS peer public key and endpoint exist.

## Mandatory gate before AD Connector

No AD Connector is created until AWS-side routed connectivity proves:

```text
MADAR-DC01 192.168.14.10
├── network reachability
├── DNS TCP/UDP 53
├── Kerberos TCP/UDP 88
├── LDAP TCP/UDP 389
├── madar.local DNS resolution
└── suitable time synchronization for Kerberos
```

This avoids paying for Directory Service while basic network prerequisites are still unverified.

## AWS account preflight already known

- target Region: `us-east-1`,
- AWS account identity verified,
- account is not currently a member of AWS Organizations,
- IAM Identity Center is not currently enabled,
- AWS credits/billing were checked before provisioning new Phase 04 resources.

## Evidence discipline

From this point forward each major gate has an explicit screenshot checkpoint. The local WireGuard gateway will be evidenced with one consolidated terminal screenshot showing hostname, static IP, persistent IPv4 forwarding, WireGuard installation/key-file permissions, DC reachability, DNS resolution, and AD TCP port checks. No private WireGuard key will be captured or committed.

## Current execution gates

```text
Gate 1  Business problem / current-state identity assessment       COMPLETE
Gate 2  VMware Windows Server + AD DS lab baseline                COMPLETE
Gate 3  Workforce users/groups + local authorization validation   COMPLETE
Gate 4  CGNAT-aware hybrid connectivity architecture              COMPLETE
Gate 5A Local MADAR-WG01 readiness                                COMPLETE
Gate 5B AWS VPC + EC2 WG-HUB + WireGuard handshake                NEXT
Gate 6  AD protocol validation from AWS side                      PENDING
Gate 7  AD Connector + IAM Identity Center                        PENDING
Gate 8  Permission sets / account assignments                     PENDING
Gate 9  SSO + MFA + temporary console/CLI sessions                PENDING
Gate 10 Positive + negative least-privilege tests                 PENDING
Gate 11 Joiner / mover / leaver lifecycle                         PENDING
Gate 12 CloudTrail audit evidence                                 PENDING
Gate 13 Cost/resource cleanup + closeout                          PENDING
```

## Runtime constraint

The local workstation has limited RAM. Local VMs will be powered on only when required:

- `MADAR-CLIENT01` stays off during tunnel construction,
- `MADAR-DC01` runs only when validating AD/DNS connectivity,
- `MADAR-WG01` is the local gateway during tunnel construction,
- all local VMs are powered off when not needed.

## Cost hold point

The architecture is designed for short-lived lab execution. Before AD Connector creation, current Directory Service pricing/free-trial eligibility must be checked again. Temporary EC2/public IPv4/Directory Service resources are deleted after evidence collection unless a later MADAR phase explicitly requires them.

## Next action

Build the Phase 04 AWS VPC and EC2 `WG-HUB`, establish the outbound WireGuard tunnel with `MADAR-WG01`, configure bidirectional routing, then validate DNS/Kerberos/LDAP connectivity to `MADAR-DC01` from the AWS side before creating any AD Connector.
