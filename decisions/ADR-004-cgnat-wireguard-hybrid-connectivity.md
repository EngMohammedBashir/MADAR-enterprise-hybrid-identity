# ADR-004 — CGNAT-Aware Hybrid Connectivity

**Status:** Accepted and implemented  
**Date:** 2026-08-25

## 🎯 Context

Phase 04 required `madar.local` to remain the corporate identity source while AWS consumed the existing directory through a real hybrid network path.

The on-premises lab runs in VMware:

- `MADAR-DC01` — Windows Server 2025, AD DS + DNS, `192.168.14.10`
- `MADAR-CLIENT01` — Windows 11 Pro domain client
- `MADAR-WG01` — Ubuntu routing endpoint, `192.168.14.30`
- local network — `192.168.14.0/24`

Preflight testing identified a key constraint: the Zain 5G connection operates behind CGNAT and does not provide a stable, directly owned public IPv4 suitable for a conventional static Customer Gateway design.

The architecture therefore needed to provide routed AWS-to-on-premises Active Directory connectivity without exposing the Domain Controller publicly or purchasing a static public IPv4 solely for a temporary portfolio lab.

## 🧭 Decision

Use a **self-managed routed WireGuard VPN** with a Linux endpoint on each side of the hybrid boundary.

```text
VMware / Home Lab                              AWS / us-east-1

MADAR-DC01                                     MADAR Phase 04 VPC
192.168.14.10                                  10.50.0.0/16
AD DS + DNS                                          |
     |                                               |
     v                                               v
MADAR-WG01  ===== encrypted WireGuard =====   EC2 WG-HUB
192.168.14.30      10.200.0.2 ↔ 10.200.0.1    10.50.1.132
                                                     |
                                                     v
                                              AD Connector
                                                     |
                                                     v
                                              Amazon WorkSpaces
                                                     |
                                                     v
                                              madar\sara.ibrahim
```

The home-side WireGuard router initiates and maintains the tunnel toward the AWS-hosted EC2 endpoint. This design works through CGNAT because unsolicited inbound connectivity to the home network is not required.

The EC2 instance operates as a network appliance. Source/destination checking is disabled, Linux IP forwarding is enabled, and AWS route tables direct the on-premises CIDR toward the WireGuard appliance.

## 💡 Why this design was selected

- Preserves `madar.local` as the authoritative corporate identity source.
- Works within the actual CGNAT and dynamic-public-address constraint.
- Keeps `MADAR-DC01` private and avoids direct Internet exposure.
- Provides real routed hybrid connectivity instead of recreating users manually in AWS.
- Supports AWS Directory Service AD Connector against the existing directory.
- Enables Amazon WorkSpaces to authenticate a real `madar.local` employee account.
- Keeps the lab infrastructure temporary and cost-conscious.
- Creates a useful engineering scenario around routing, DNS, Kerberos, directory integration, failure isolation and recovery.

## 🏗️ Implementation characteristics

### On-premises endpoint

`MADAR-WG01` provides the local routed VPN endpoint between `192.168.14.0/24` and the WireGuard transit network.

### AWS endpoint

The EC2 WG-HUB provides the AWS-side routing endpoint. During the completed lab run:

- EC2: `i-029deb16c4c36fd11`
- private IP: `10.50.1.132`
- Elastic IP: `34.228.95.241`
- WireGuard AWS address: `10.200.0.1`

These identifiers document the completed run and are not architectural constants; a rebuild will normally generate new resource IDs and addresses.

### Directory integration

AWS Directory Service AD Connector (`d-90667da553` during the completed run) consumed the existing `madar.local` directory over the routed hybrid path.

### Workforce validation

Amazon WorkSpaces provided the final end-user authentication proof. The domain user `madar\sara.ibrahim` successfully authenticated to WorkSpace `ws-49q8s94dl`, which joined `MADAR.LOCAL` as computer `WSAMZN-I0F8R2FL` with private IP `10.50.13.89`.

## 🔐 Security positioning

This architecture is **not** represented as AWS managed Site-to-Site VPN.

It is a **self-managed WireGuard routed VPN running through an EC2 network appliance**, deliberately selected for the constraints of the lab environment.

Security controls include:

- no public exposure of the Domain Controller,
- scoped network paths between AWS and the on-premises AD network,
- WireGuard encrypted transport,
- no requirement to duplicate corporate employee identities in AWS,
- Windows Firewall retained as part of the normal design rather than globally disabled,
- temporary AWS networking resources removed after evidence collection.

For a production enterprise environment, an organization-approved managed VPN/firewall platform, AWS Site-to-Site VPN, Direct Connect, or another connectivity architecture with appropriate availability and support guarantees would normally be evaluated.

## ⚖️ Alternatives considered

### AWS Site-to-Site VPN with a static Customer Gateway

Not selected for this lab because the home connection does not provide a stable, directly owned public IPv4 suitable for the intended Customer Gateway design.

### AWS Client VPN

Not selected because the requirement is network-to-network reachability from AWS workloads and directory services toward the on-premises AD network, rather than remote-user access into a VPC.

### Duplicate cloud-native users

Rejected because creating unrelated AWS-side copies of corporate users would bypass the central identity objective and introduce duplicate lifecycle management.

### AWS Managed Microsoft AD / forest trust

Not selected as a connectivity workaround because directory trust and integration still require network line-of-sight and would change the directory ownership model being demonstrated.

### Entra ID / external IdP federation

A valid alternative for a different architecture, particularly where outbound identity synchronization or federation is preferred. It was outside this lab's objective of proving routed integration with the existing Active Directory environment.

## 🚦 Mandatory validation gates

AD Connector and WorkSpaces should not be treated as validated until the AWS-side path can demonstrate the required directory connectivity, including:

- reachability to `192.168.14.10`,
- DNS TCP/UDP 53,
- Kerberos TCP/UDP 88,
- LDAP TCP/UDP 389,
- correct `madar.local` DNS resolution,
- acceptable time synchronization for Kerberos,
- successful AD Connector registration,
- successful domain-user authentication from Amazon WorkSpaces.

## 🧪 Failure and recovery validation

The hybrid path was deliberately interrupted and restored.

Expected dependency chain:

```text
WorkSpace
   ↓
AWS routing
   ↓
EC2 WG-HUB
   ↓
WireGuard tunnel
   ↓
MADAR-WG01
   ↓
MADAR-DC01 / DNS / AD DS
```

With the WireGuard path unavailable, WorkSpace-to-DC DNS/TCP 53 connectivity failed. After the tunnel was restored and a fresh WireGuard handshake was established, connectivity and `madar.local` DNS resolution recovered.

This demonstrated that the hybrid identity path had a real network dependency rather than being a diagram-only integration.

## 💰 Cost and cleanup guardrail

The WireGuard hub, Elastic IP, AD Connector and WorkSpace are temporary lab infrastructure.

Closeout includes:

1. terminate the WorkSpace,
2. deregister WorkSpaces from the directory when applicable,
3. delete the AD Connector,
4. disassociate and release the Elastic IP,
5. terminate the EC2 WG-HUB,
6. remove Phase 04-only routes, security rules and networking resources,
7. verify billing/Cost Explorer for unintended residual spend.

Local VMware systems may be powered off and retained for later portfolio phases where they continue to provide value.

## 📊 Consequences

### Positive

- Solves the actual CGNAT constraint without weakening the identity objective.
- Keeps the Domain Controller private.
- Demonstrates genuine routed hybrid connectivity.
- Produces a strong troubleshooting and failure-recovery story.
- Reuses the existing corporate identity source.
- Keeps the cloud-side lab footprint temporary.

### Trade-offs

- WireGuard is self-managed rather than an AWS-managed VPN service.
- Routing, patching and tunnel configuration remain operator responsibilities.
- The EC2 network appliance becomes a connectivity dependency while the lab is active.
- Directory authentication depends on healthy DNS, routing and tunnel state.
- This design is appropriate for the portfolio lab but is not presented as the default production enterprise connectivity architecture.

## ✅ Observed success criteria

The implementation was considered successful after all of the following were observed:

```text
Encrypted WireGuard tunnel                  PASS
AWS → on-premises AD/DNS connectivity       PASS
AD Connector against madar.local            PASS
WorkSpace domain join                       PASS
madar\sara.ibrahim authentication           PASS
WorkSpace → DC DNS/TCP 53                    PASS
Intentional tunnel failure                  OBSERVED
Connectivity failure during outage          OBSERVED
Tunnel restoration / fresh handshake        PASS
DNS/connectivity recovery                    PASS
```

The implemented result therefore validates the architecture as **hybrid identity consumption through AD Connector and Amazon WorkSpaces**, while direct AWS-account SSO through IAM Identity Center remains a separate future production extension.