# MADAR — Enterprise Identity & Workforce Access

## Phase 04 of the MADAR Cloud Transformation

> **Status: ACTIVE — LOCAL IDENTITY VALIDATED / HYBRID CONNECTIVITY BUILD NEXT**  
> Corporate Active Directory representation → local authorization proof → CGNAT-aware routed WireGuard connectivity → AD Connector → IAM Identity Center → SSO/MFA → permission sets → lifecycle/audit validation → cleanup.

MADAR has completed the representative workload migration in Phase 03. Phase 04 addresses the next business problem: **how employees securely access the growing AWS environment through centralized workforce identity instead of individual long-lived IAM credentials.**

## Current execution state

The local corporate-identity foundation is built and validated.

```text
VMware lab
│
├── MADAR-DC01
│   ├── Windows Server 2025
│   ├── Active Directory Domain Services
│   ├── DNS
│   ├── Domain: madar.local
│   ├── Department OUs
│   ├── Security groups
│   └── Group Policy
│
├── MADAR-CLIENT01
│   ├── Windows 11 Pro
│   ├── joined to madar.local
│   ├── domain-user authentication validated
│   └── GPO enforcement validated
│
└── MADAR-LEGACY-01
    └── retained representative legacy workload
```

Validated locally:

- `MADAR-DC01` promoted to a domain controller for `madar.local`,
- five synthetic department identities created,
- group membership verified with PowerShell,
- `MADAR-CLIENT01` successfully joined to the domain,
- domain login validated with `sara.ibrahim`,
- `GPO-IT-Security` applied to the client,
- Domain firewall policy validated,
- `GG-IT` access to the IT share succeeded,
- cross-department access to the Finance share was denied as designed.

## Hybrid-connectivity constraint discovered

Preflight testing of the Zain 5G home-lab connection established that:

- the router WAN IPv4 is private (`10.x.x.x`),
- the lab sits behind carrier-grade NAT,
- the observed public IPv4 changes after reconnect/restart,
- the home lab therefore does not provide the stable directly-owned public IPv4 expected by the classic static-address Customer Gateway design.

The project will not pretend this constraint does not exist and will not expose the domain controller publicly.

## Selected lab architecture

The accepted Phase 04 lab design uses a **self-managed routed WireGuard VPN** initiated outbound from the home lab and terminated on an EC2 network appliance.

```text
HOME / VMware                                  AWS / us-east-1

MADAR-DC01                                     Phase 04 VPC
192.168.14.10                                  non-overlapping CIDR
AD DS + DNS                                         |
      |                                             |
      v                                             v
MADAR-WG01   ===== WireGuard tunnel =====      EC2 WG-HUB
Linux router       outbound initiated          Linux network appliance
      |                                             |
192.168.14.0/24                                    |
                                                    v
                                              AD Connector
                                                    |
                                                    v
                                           IAM Identity Center
                                                    |
                                                    v
                                     Groups + Permission Sets
                                                    |
                                                    v
                                      SSO + MFA + CLI + Audit
```

Why this design:

- keeps `madar.local` as the central corporate identity source,
- solves the lab's CGNAT/dynamic-public-IP constraint by initiating the tunnel outbound,
- creates a real routed path instead of recreating workforce users manually in AWS,
- keeps `MADAR-DC01` private,
- is inexpensive and temporary enough for a portfolio lab,
- creates a defensible architecture decision around a real networking constraint.

Important: this is **not** described as AWS managed Site-to-Site VPN. It is a **self-managed WireGuard routed VPN using an EC2 network appliance**.

Architecture decision: [`decisions/ADR-004-cgnat-wireguard-hybrid-connectivity.md`](decisions/ADR-004-cgnat-wireguard-hybrid-connectivity.md)  
Execution plan: [`docs/12-wireguard-hybrid-execution-plan.md`](docs/12-wireguard-hybrid-execution-plan.md)

## Mandatory gate before AD Connector

AD Connector is not created until the AWS-side routed path proves the required local AD services are reachable:

```text
MADAR-DC01 — 192.168.14.10
├── network reachability
├── DNS TCP/UDP 53
├── Kerberos TCP/UDP 88
├── LDAP TCP/UDP 389
├── madar.local DNS resolution
└── suitable time synchronization for Kerberos
```

This prevents Directory Service spend while routing/DNS/firewall problems are unresolved.

## Business story

MADAR's company narrative assumes a corporate employee directory existed before Phase 04. The VMware-hosted Windows Server environment is a **reproducible lab representation** of that existing identity system; it is not a second workload migration and does not rewrite the Phase 03 chronology.

```text
Corporate workforce
      ↓
MADAR Active Directory
      ↓
Routed hybrid connectivity
      ↓
AWS Directory integration
      ↓
AWS IAM Identity Center
      ↓
Groups + Permission Sets
      ↓
SSO + MFA
      ↓
Temporary Console / CLI sessions
```

## Local workforce model

| Department | Synthetic employee | AD security group |
|---|---|---|
| Management | Ahmed Ali (`ahmed.ali`) | `GG-Management` |
| IT | Sara Ibrahim (`sara.ibrahim`) | `GG-IT` |
| Finance | Omar Hassan (`omar.hassan`) | `GG-Finance` |
| HR | Noura Saleh (`noura.saleh`) | `GG-HR` |
| Sales | Khalid Mansour (`khalid.mansour`) | `GG-Sales` |

These identities are synthetic and exist only for the lab.

## Phase objective

Phase 04 closes only after the local identity source is integrated with AWS and the project demonstrates:

- centralized workforce identity,
- group-based authorization,
- IAM Identity Center,
- SSO and MFA,
- temporary AWS credentials,
- permission sets and least privilege,
- positive and negative AWS access tests,
- console and CLI SSO,
- onboarding, role-change and offboarding workflows,
- audit evidence,
- cost-aware cleanup.

## Execution sequence

```text
Local AD/client validation                       COMPLETE
      ↓
CGNAT/public-IP preflight                        COMPLETE
      ↓
WireGuard architecture decision                  COMPLETE
      ↓
MADAR-WG01 + AWS WG-HUB                          NEXT
      ↓
Bidirectional routing + AD protocol validation
      ↓
AD Connector
      ↓
AWS Organizations / IAM Identity Center
      ↓
Permission Sets + account assignments
      ↓
SSO + MFA + temporary sessions
      ↓
AWS CLI SSO
      ↓
Positive + negative authorization tests
      ↓
Joiner / mover / leaver
      ↓
CloudTrail audit
      ↓
Evidence + cost/resource cleanup
      ↓
PHASE 04 ACCEPTED
```

The detailed, no-skip execution checklist lives in [`checklists/phase04-master-checklist.md`](checklists/phase04-master-checklist.md).

## Evidence highlights

### Domain controller verification

![Domain controller verification](evidence/Domain-Controller-Verification.png)

### Workforce security-group verification

![AD security group membership verification](evidence/AD-Security-Group-Membership-Verification.png)

### Domain join

![CLIENT01 domain join success](evidence/CLIENT01-Domain-Join-Success.png)

### GPO applied

![GPO IT Security applied](evidence/GPO-IT-Security-Applied.png)

### Least-privilege local access test

Allowed IT share access:

![Sara IT share access success](evidence/Sara-IT-Share-Access-Success.png)

Denied Finance share access:

![Sara Finance access denied](evidence/Sara-Finance-Access-Denied.png)

See [`evidence/README.md`](evidence/README.md) for the complete evidence index.

## Execution style

```text
New concept         → GUI / Console first
Understand resource → PowerShell / CLI inspection
Repeated operation  → automation where useful
Validation          → command output + visual evidence
Security boundary   → explicit negative tests
```

## Cost guardrail

The local VMware identity lab itself does not consume AWS resources. The selected connectivity path adds temporary AWS EC2/public-IPv4 usage and later Directory Service usage.

Rules:

- current pricing/free-trial eligibility is checked immediately before AD Connector creation,
- paid AWS resources are created only when the preceding technical gate has passed,
- evidence is collected promptly,
- temporary EC2/public-IP/Directory Service resources are removed after acceptance unless explicitly required by a later phase,
- final Bills / Cost Explorer and resource inventory are reviewed before Phase 04 is marked complete.

## Repository structure

```text
.
├── README.md
├── CURRENT-STATE.md
├── REPOSITORY-SCOPE.md
├── checklists/
├── decisions/
├── docs/
├── evidence/
├── identity-model/
├── policies/
├── runbooks/
└── tests/
```

## Relationship to the MADAR journey

```text
Phase 01  Cloud Foundation                     COMPLETE
Phase 02  Serverless Event Processing          COMPLETE
Phase 03  Legacy Migration & Data Center Exit  COMPLETE
Phase 04  Enterprise Identity & Workforce      ACTIVE
          ├── Local AD/client validation        COMPLETE
          ├── Hybrid design                     COMPLETE
          └── WireGuard/AWS integration         NEXT
Phase 05  Application Modernization            FUTURE
```

Master transformation record: [`EngMohammedBashir/MADAR-cloud-transformation`](https://github.com/EngMohammedBashir/MADAR-cloud-transformation)
