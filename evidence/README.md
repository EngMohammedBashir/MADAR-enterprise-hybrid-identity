# Phase 04 Evidence Index

This folder contains evidence that proves an identity, policy, connectivity or authorization claim. Screenshots use descriptive filenames and are linked from the project README where they add value.

> Evidence is treated as proof, not decoration. Each screenshot should answer a specific engineering question.

## 1 — Local identity / Active Directory evidence

| Evidence file | What it proves |
|---|---|
| `legacy-vm-running-before-identity-modernization.png` | representative legacy environment remained available before identity modernization work |
| `Domain-Controller-Verification.png` | `MADAR-DC01`, `madar.local` and domain-controller role verification |
| `Active-Directory-Domain-Controller.png` | `MADAR-DC01` visible in Active Directory as the domain controller |
| `Active-Directory-OU-Structure.png` | MADAR departmental OU structure |
| `Active-Directory-Security-Groups.png` | Global Security Groups for workforce departments |
| `Manual-AD-User-Creation.png` | first synthetic user created manually to understand the GUI workflow |
| `PowerShell-Automated-AD-Users-Creation.png` | repeat user creation validated through PowerShell automation |
| `AD-Security-Group-Membership-Verification.png` | each synthetic user mapped to the intended departmental security group |
| `GPO-IT-Domain-Firewall-Policy.png` | configured IT Domain firewall policy |
| `CLIENT01-System-Verification.png` | Windows client baseline / hostname verification |
| `CLIENT01-Domain-Join-Success.png` | `MADAR-CLIENT01` successfully joined `madar.local` |
| `Domain-User-Login-Sara.png` | domain-user authentication on the client |
| `GPO-IT-Security-Applied.png` | `GPO-IT-Security` applied to the domain client |
| `Domain-Firewall-GPO-Verification.png` | Domain firewall enforcement verified on the client |
| `Sara-IT-Share-Access-Success.png` | intended IT-share access succeeded for the IT identity |
| `Sara-Finance-Access-Denied.png` | cross-department Finance-share access was denied as designed |

### Local identity story

```text
Domain Controller
      ↓
OU + Users + Groups
      ↓
Domain Client Join
      ↓
Domain User Login
      ↓
GPO Enforcement
      ↓
Allowed IT Access ✅
Denied Finance Access ❌
```

The denied-access screenshot is a **success condition**: it proves that group membership produces a real authorization boundary.

## 2 — Hybrid connectivity evidence

| Evidence file | What it proves |
|---|---|
| `MADAR-WG01-Local-PreAWS-Validation.png` | local Linux gateway readiness before AWS integration |
| `AWS-VPC-Subnets-Validation.png` | dedicated Phase 04 VPC/private-subnet foundation |
| `AWS-Route-Tables-Validation.png` | AWS routing configuration used for the hybrid path |
| `phase04-wireguard-tunnel-evidence.png` | encrypted WireGuard peer relationship / tunnel operation |
| `phase04-aws-to-onprem-ad-connectivity-evidence.png` | AWS-side path reaches the on-premises AD/DNS environment |

### Hybrid connectivity story

```text
MADAR-DC01 192.168.14.10
        ↑
        │ local LAN
        │
MADAR-WG01 192.168.14.30
        ↑
        ║ encrypted WireGuard
        ↓
AWS WG-HUB / EC2
        ↑
        │ VPC routing
        │
AD Connector private subnets
```

The important proof is end-to-end behavior. A WireGuard handshake alone proves only that the peers can establish a tunnel; it does **not** prove that AWS workloads can route through that tunnel to Active Directory.

During troubleshooting, packet capture was used to prove DNS TCP/53 request/reply traffic between the AWS side and `MADAR-DC01`. That evidence separated the network layer from the later credential failure.

## 3 — Troubleshooting evidence narrative

The AD Connector initially reported:

```text
DNS unavailable (TCP port 53) for 192.168.14.10
```

The investigation validated the route-table association, appliance target, Directory Service ENIs, EC2 Security Groups, Linux forwarding/NAT and packet flow. The AWS hub Security Group originally admitted WireGuard UDP/51820 but not VPC transit traffic. After the VPC-side path was corrected, DNS traffic reached the domain controller and replies returned.

The next connector attempt progressed to:

```text
Invalid credentials (bad username/password)
```

That change is useful evidence: the network barrier was removed and the integration reached the authentication layer. The project is intentionally retaining this troubleshooting story because it demonstrates layered diagnosis rather than blind resource recreation.

## 4 — AWS identity evidence still pending

The next evidence set should cover:

```text
AD Connector Active
        ↓
IAM Identity Center identity source
        ↓
Permission sets + account assignments
        ↓
SSO + MFA
        ↓
Temporary console / CLI session
        ↓
Allowed AWS action ✅
Denied AWS action ❌
        ↓
Joiner / Mover / Leaver
        ↓
CloudTrail audit
        ↓
Final cost + cleanup audit
```

## Evidence rules

- Never expose passwords, MFA seeds, private keys, reusable access keys or session tokens.
- Do not commit WireGuard private keys.
- Prefer one strong screenshot that proves a gate over many repetitive screenshots.
- Keep intentional failures when they prove least privilege or document a meaningful troubleshooting decision.
- Crop/compose screenshots so a reviewer can understand the claim quickly.
- Lab passwords are intentionally excluded from repository documentation.