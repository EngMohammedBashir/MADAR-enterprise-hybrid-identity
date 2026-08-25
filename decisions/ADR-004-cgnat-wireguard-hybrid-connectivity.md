# ADR-004 — CGNAT-aware hybrid connectivity for Phase 04

**Status:** Accepted for the lab  
**Date:** 2026-08-25

## Context

Phase 04 requires `madar.local` to remain the corporate identity source while AWS IAM Identity Center consumes identities through an Active Directory integration path.

The local environment runs in VMware:

- `MADAR-DC01` — Windows Server 2025, AD DS + DNS, `192.168.14.10`
- `MADAR-CLIENT01` — Windows 11 Pro domain client
- local network — `192.168.14.0/24`

The Internet connection is Zain 5G. Preflight testing established:

- router WAN IPv4 is private (`10.x.x.x`), indicating carrier NAT between the router and the public Internet,
- the public IPv4 changed after router restart/reconnect,
- therefore the lab does not have a directly owned, stable public IPv4 suitable for the classic static-address Customer Gateway design.

The requirement is to preserve a real routed path between the AWS VPC and `MADAR-DC01` without exposing the domain controller publicly and without buying a static public IPv4 solely for a short-lived portfolio lab.

## Decision

Use a **self-managed routed WireGuard VPN** with two Linux routing endpoints:

```text
VMware / Home Lab                         AWS / us-east-1

MADAR-DC01                                Phase 04 VPC
192.168.14.10                             non-overlapping CIDR
     |                                         |
     v                                         v
MADAR-WG01  ===== WireGuard tunnel =====  EC2 WG-HUB
Linux router      outbound initiated       Linux network appliance
     |                                         |
192.168.14.0/24                         route back to 192.168.14.0/24
                                               |
                                               v
                                         AD Connector
                                               |
                                               v
                                       IAM Identity Center
```

The local WireGuard router initiates the tunnel outbound to an AWS-hosted EC2 endpoint. This avoids requiring unsolicited inbound reachability to the home network and tolerates the home-side public IPv4 changing while the peer session is re-established.

The EC2 instance acts as a network appliance. Its source/destination check must be disabled, IP forwarding must be enabled, and the private-subnet route table must direct the local AD CIDR toward the appliance path.

## Why this decision was selected

- Preserves `madar.local` as the corporate identity source.
- Works with the lab's CGNAT/dynamic-address constraint because the home side initiates the tunnel outbound.
- Avoids exposing `MADAR-DC01` directly to the Internet.
- Provides real routed hybrid connectivity rather than recreating users manually in AWS.
- Uses inexpensive, short-lived resources appropriate for a portfolio lab.
- Demonstrates routing, NAT constraints, network appliances, Active Directory reachability, IAM Identity Center, and least-privilege identity design.

## Important positioning

This is **not** documented as AWS managed Site-to-Site VPN.

It is a **self-managed WireGuard routed VPN on an EC2 network appliance**, selected because the home-lab Internet connection is behind CGNAT with a changing public IPv4.

For production, a managed Site-to-Site VPN, enterprise firewall/VPN appliance integration, Direct Connect, or another organization-approved connectivity service would normally be preferred when the required endpoints and service guarantees are available.

## Alternatives considered

### Classic AWS Site-to-Site VPN with static Customer Gateway address

Rejected for this lab because the home connection does not provide a stable, directly owned public IPv4.

### AWS Client VPN

Rejected because the project requires routed site/network connectivity from the AWS VPC toward the local AD network, not simply remote-user access into a VPC.

### Recreate users directly in IAM Identity Center

Rejected because it would bypass the central project requirement to demonstrate centralized workforce identity sourced from the existing `madar.local` directory.

### AWS Managed Microsoft AD / forest trust

Not selected as a network workaround because trust/integration still requires network line-of-sight to the local directory and changes the ownership/model of the directory architecture.

### Entra ID synchronization / external IdP federation

Valid alternative architecture, especially when outbound-only identity synchronization is preferred, but not selected because the current Phase 04 objective is to demonstrate a routed Active Directory integration path into AWS.

## Mandatory implementation gates

No AD Connector is created until all of the following are proven from the AWS-side routed path:

- reachability to `192.168.14.10`,
- DNS TCP/UDP 53,
- Kerberos TCP/UDP 88,
- LDAP TCP/UDP 389,
- correct `madar.local` DNS resolution,
- acceptable time synchronization for Kerberos.

The Windows Firewall must not be globally disabled as the normal design. Required rules should be scoped to the AWS/VPN source networks.

## Cost and cleanup guardrail

The WireGuard components are temporary lab infrastructure.

Before creating Directory Service resources, pricing/free-trial eligibility must be checked again. At closeout:

- delete AD Connector if no longer required,
- terminate the EC2 WireGuard hub,
- release any public IPv4 / Elastic IP allocation that should not persist,
- remove temporary routes/security rules,
- verify billing and Cost Explorer show no unintended Phase 04 ongoing resources.

## Consequences

### Positive

- solves the actual CGNAT constraint without weakening the identity objective,
- keeps the domain controller private,
- offers strong portfolio/interview discussion around constraints and trade-offs,
- low resource footprint for the local 8 GB workstation.

### Negative

- the VPN is self-managed rather than a managed AWS VPN service,
- routing and WireGuard configuration become our responsibility,
- the EC2 network appliance becomes a temporary connectivity dependency,
- additional troubleshooting is required before Directory Service creation.

## Success criteria

The decision is successful when an AWS-side test path can reach required AD services on `MADAR-DC01`, AD Connector becomes Active, IAM Identity Center can use the intended AD identity source, and the resulting workforce SSO/permission/lifecycle tests complete without exposing the domain controller publicly.
