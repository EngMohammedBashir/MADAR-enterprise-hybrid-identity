# Phase 04 — WireGuard Hybrid Execution Plan

This document is the implementation handoff for the AWS portion of Phase 04. It exists to prevent missed prerequisites, out-of-order resource creation, unnecessary cost, and repeated VM start/stop cycles.

## Locked lab architecture

```text
HOME / VMware                                 AWS / us-east-1

MADAR-DC01                                    Phase 04 VPC
192.168.14.10                                 non-overlapping CIDR
AD DS + DNS                                        |
     |                                             |
     v                                             v
MADAR-WG01  ===== encrypted WireGuard =====   EC2 WG-HUB
Linux router       outbound initiated          Linux network appliance
     |                                             |
     +---- 192.168.14.0/24                        +---- route back to home CIDR
                                                   |
                                                   v
                                             AD Connector
                                                   |
                                                   v
                                           IAM Identity Center
                                                   |
                                                   v
                                      Groups -> Permission Sets
                                                   |
                                                   v
                                    SSO / MFA / CLI / CloudTrail
```

## Naming plan

Local:

- `MADAR-DC01` — existing domain controller, `192.168.14.10`
- `MADAR-CLIENT01` — existing Windows 11 domain client
- `MADAR-WG01` — new lightweight Linux WireGuard router

AWS:

- VPC: `MADAR-P04-VPC`
- public subnet: `MADAR-P04-PUBLIC-A`
- private subnet A: `MADAR-P04-PRIVATE-A`
- private subnet B: `MADAR-P04-PRIVATE-B`
- Internet Gateway: `MADAR-P04-IGW`
- EC2: `MADAR-P04-WG-HUB`
- Security Group: `MADAR-P04-WG-SG`
- AD Connector: `MADAR-P04-AD-CONNECTOR`

Exact CIDRs and AZs must be chosen only after confirming they do not overlap `192.168.14.0/24` or any other network that must route through the tunnel.

## Execution rules

1. Do not create AD Connector before routed AD protocol tests pass.
2. Do not disable the Windows Firewall globally as the normal solution.
3. Do not call the self-managed WireGuard design `AWS Site-to-Site VPN` in documentation.
4. Do not expose `MADAR-DC01` directly to the public Internet.
5. Do not leave EC2, public IPv4, or Directory Service resources running after acceptance unless explicitly required.
6. Capture evidence at success gates before moving on.
7. Keep `MADAR-CLIENT01` powered off until an end-user SSO test actually needs it.

## Stage 1 — Local `MADAR-WG01`

### Build

- create a minimal Ubuntu Server Linux VM,
- allocate only the RAM needed for routing/WireGuard,
- attach it to the same VMware network as `MADAR-DC01`,
- identify an unused IP on `192.168.14.0/24`,
- configure that IP statically,
- verify it can reach `192.168.14.10`,
- install WireGuard,
- enable `net.ipv4.ip_forward=1`.

### Hold point

Before configuring Windows routes, verify the chosen `MADAR-WG01` IPv4 is correct and stable.

## Stage 2 — AWS network

### Build

- use `us-east-1`,
- create a dedicated non-overlapping VPC,
- create one public subnet for `WG-HUB`,
- create two private subnets in different AZs for AD Connector,
- attach an Internet Gateway,
- configure the public route table for Internet access,
- keep AD Connector subnets private,
- create the Security Group for the WireGuard appliance.

### Evidence

Capture VPC/subnet/route architecture only after final routing is correct.

## Stage 3 — EC2 `WG-HUB`

### Build

- launch small Ubuntu EC2 in the public subnet,
- assign a public IPv4 / Elastic IP as required by the chosen lab setup,
- install WireGuard,
- enable IPv4 forwarding,
- disable EC2 source/destination check,
- configure Security Group inbound for the selected WireGuard UDP port,
- restrict other inbound access,
- configure forwarding policy only for required routed networks.

### Critical failure checks

If routed traffic does not pass, verify:

- source/destination check is disabled,
- Linux IP forwarding is enabled,
- WireGuard peer AllowedIPs are correct,
- Security Group/NACL do not block required traffic,
- route-table targets are correct.

## Stage 4 — WireGuard tunnel

### Local peer behavior

`MADAR-WG01` initiates outbound toward AWS. Configure `PersistentKeepalive` so the CGNAT mapping remains usable.

### Required proof

- WireGuard handshake succeeds,
- tunnel counters increase in both directions,
- the AWS peer can reach the local router's tunnel-side path,
- the local router can reach AWS VPC addresses through the tunnel.

Capture handshake/routing evidence here.

## Stage 5 — Bidirectional network routing

### AWS side

The private-subnet route table used by AD Connector must contain a route for:

`192.168.14.0/24` -> EC2 network-appliance path / ENI target as implemented.

### Local side

`MADAR-DC01` needs a persistent route for the AWS VPC CIDR through `MADAR-WG01`.

Do not hard-code the next hop in documentation until the actual unused static IP assigned to `MADAR-WG01` is known.

### Required proof

Traffic must return through the same routed tunnel path. Verify both directions rather than relying on one successful ping.

## Stage 6 — AD protocol validation

Before AD Connector creation, prove from the AWS-side routed path:

- network reachability to `192.168.14.10`,
- DNS TCP/UDP 53,
- Kerberos TCP/UDP 88,
- LDAP TCP/UDP 389,
- `madar.local` resolves using the local AD DNS,
- time is sufficiently synchronized for Kerberos.

Only add additional AD ports if the actual integration/test requires them. Avoid unnecessarily broad rules.

### STOP condition

If any core AD protocol test fails, do not create AD Connector. Fix routing/DNS/firewall first.

## Stage 7 — AWS Organizations / IAM Identity Center prerequisites

Before final identity-source configuration:

- confirm current Organizations state,
- create/configure the Organization only when required by the selected IAM Identity Center model,
- confirm the account that must own the directory integration,
- enable IAM Identity Center in `us-east-1`,
- verify the region/account relationship before connector assignment.

## Stage 8 — AD Connector

Immediately before creation:

- check current Directory Service pricing/free-trial eligibility,
- confirm the selected two private subnets are in different AZs,
- confirm `madar.local` and DNS server `192.168.14.10`,
- use credentials appropriate for AD Connector setup without exposing secrets in GitHub/screenshots,
- create the connector,
- wait for `Active`.

If creation fails, troubleshoot using the already-proven network facts rather than recreating resources blindly.

## Stage 9 — IAM Identity Center integration

- select the intended AD/Directory Service identity source,
- verify MADAR workforce identities/groups can be used,
- validate the AWS access portal,
- configure MFA according to the final selected Identity Center strategy,
- confirm no workforce user requires a long-lived IAM access key.

## Stage 10 — Permission model

Finalize the mapping before creating assignments.

Required permission sets:

- Cloud Admin
- DevOps
- Developer
- Security
- Auditor

Use groups for assignment wherever practical.

## Stage 11 — Positive and negative tests

Required positive proof:

- Cloud Admin allowed action,
- DevOps allowed operation,
- Developer allowed application/development operation,
- Security allowed read/investigation action,
- Auditor read-only action.

Required negative proof:

- Developer IAM-admin action denied,
- Auditor write/delete denied,
- Security infrastructure-admin action denied.

An intentional `AccessDenied` result is success evidence when it proves least privilege.

## Stage 12 — SSO and CLI

- validate browser SSO,
- validate MFA,
- validate temporary sessions,
- configure AWS CLI SSO,
- run `aws sso login`,
- verify CLI caller identity/session uses temporary SSO credentials rather than long-lived workforce keys.

## Stage 13 — Identity lifecycle

Demonstrate:

- Joiner — new/enable identity -> correct group -> access appears,
- Mover — group/role change -> permissions change,
- Leaver — disable/remove identity -> future access revoked.

Capture proof of revocation, not only the configuration change.

## Stage 14 — Audit

Use CloudTrail Event History to locate:

- workforce/SSO principal,
- allowed action,
- denied action where available,
- temporary-session activity.

Evidence must avoid exposing session tokens or secrets.

## Stage 15 — Documentation / portfolio

Update:

- architecture diagram,
- ADR,
- `README.md`,
- `CURRENT-STATE.md`,
- master checklist,
- evidence index,
- master MADAR transformation repository.

Explain the CGNAT constraint and the decision to use a self-managed outbound WireGuard path honestly.

## Stage 16 — Cleanup

Delete/terminate/release as appropriate:

- AD Connector,
- EC2 `WG-HUB`,
- public IPv4 / Elastic IP,
- temporary AWS routing/security resources,
- any other paid Phase 04 resource no longer required.

Then:

- power off local VMs,
- check Bills / Cost Explorer,
- verify no unintended Phase 04 paid resources remain,
- record the closeout decision.

## Screenshot / evidence reminders

Capture evidence at these points:

1. CGNAT/public-IP preflight outcome (if safe and useful; no router secrets),
2. WireGuard handshake,
3. AWS-to-AD routed/protocol validation,
4. AD Connector Active,
5. IAM Identity Center identity source,
6. permission sets,
7. account assignments,
8. SSO success,
9. MFA proof,
10. CLI SSO proof,
11. allowed action,
12. denied action,
13. lifecycle/offboarding revocation,
14. CloudTrail audit,
15. final architecture,
16. final cost/resource cleanup.

## Final acceptance

Phase 04 is complete only when:

```text
Corporate AD identity
        + routed hybrid connectivity
        + IAM Identity Center SSO/MFA
        + group-based temporary AWS permissions
        + positive authorization proof
        + negative authorization proof
        + CLI SSO
        + lifecycle revocation
        + CloudTrail audit
        + cleanup verification
        = ACCEPTED
```
