# Phase 04 Evidence Index

This folder contains evidence that proves an identity, policy, connectivity, authentication or recovery claim.

> Evidence is treated as proof, not decoration. Each screenshot should answer one engineering question.

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
| `Domain-User-Login-Sara.png` | local domain-user authentication on the client |
| `GPO-IT-Security-Applied.png` | `GPO-IT-Security` applied to the domain client |
| `Domain-Firewall-GPO-Verification.png` | Domain firewall enforcement verified on the client |
| `Sara-IT-Share-Access-Success.png` | intended IT-share access succeeded for the IT identity |
| `Sara-Finance-Access-Denied.png` | cross-department Finance-share access was denied as designed |

## 2 — Hybrid connectivity evidence

| Evidence file | What it proves |
|---|---|
| `MADAR-WG01-Local-PreAWS-Validation.png` | local Linux gateway readiness before AWS integration |
| `AWS-VPC-Subnets-Validation.png` | dedicated Phase 04 VPC/private-subnet foundation |
| `AWS-Route-Tables-Validation.png` | AWS routing configuration used for the hybrid path |
| `phase04-wireguard-tunnel-evidence.png` | encrypted WireGuard peer relationship / tunnel operation |
| `phase04-aws-to-onprem-ad-connectivity-evidence.png` | AWS-side path reaches the on-premises AD/DNS environment |

A WireGuard handshake alone proves only peer connectivity. The routed application path was validated separately with AWS-originated AD/DNS traffic and packet capture.

## 3 — Directory Service evidence

| Evidence file | What it proves |
|---|---|
| `ad-connector-active-success.png` | AWS Directory Service AD Connector for `madar.local` reached `Active` |

The Connector troubleshooting story progressed through two distinct layers:

```text
DNS unavailable
      ↓
fix VPC transit / routing / forwarding path
      ↓
Invalid credentials
      ↓
validate/reset svc-adconnector
      ↓
AD Connector Active ✅
```

## 4 — Amazon WorkSpaces end-to-end hybrid identity evidence

| Evidence file | What it proves |
|---|---|
| `workspaces-onprem-ad-domain-join-verified.png` | AWS WorkSpace computer object exists in the on-prem `WorkSpaces` OU with `madar.local` DNS hostname |
| `workspaces-hybrid-ad-authentication-validation.png` | `madar\sara.ibrahim` authenticated successfully inside the AWS WorkSpace |
| `workspaces-to-onprem-baseline-connectivity.png` | WorkSpace `10.50.13.89` reached `MADAR-DC01` TCP/53 and resolved `madar.local` before failure injection |
| `workspaces-to-onprem-vpn-failure-test.png` | stopping WireGuard caused the expected TCP/53 failure and DNS timeout from the WorkSpace |
| `workspaces-vpn-failure-recovery-validation.png` | after restoring WireGuard, WorkSpace TCP/DNS connectivity to the on-prem domain recovered |

### WorkSpaces proof chain

```text
On-prem AD user: sara.ibrahim
        ↓
AD Connector Active
        ↓
WorkSpaces directory registered
        ↓
AWS WorkSpace created
        ↓
Computer object created in on-prem AD
        ↓
User logs in as madar\sara.ibrahim
        ↓
Healthy hybrid DNS path verified
        ↓
VPN failure injected
        ↓
Connectivity loss observed
        ↓
VPN restored
        ↓
Connectivity recovery verified
```

## 5 — What is intentionally not claimed

The original Phase 04 design included IAM Identity Center, SSO/MFA, permission sets and CLI SSO. Those were **not implemented** in this Free Plan lab because the account was not upgraded and Organizations changes were not forced solely to satisfy the initial design.

Therefore this evidence folder does not claim screenshots for:

- IAM Identity Center SSO,
- MFA through Identity Center,
- permission-set assignments,
- CLI SSO,
- CloudTrail SSO principal activity.

Those remain future production-extension evidence targets.

## Evidence rules

- Never expose passwords, MFA seeds, private keys, reusable access keys or session tokens.
- Do not commit WireGuard private keys.
- Prefer one strong screenshot that proves a gate over many repetitive screenshots.
- Keep intentional failures when they prove a dependency or least-privilege boundary.
- Crop/compose screenshots so a reviewer can understand the claim quickly.
- Lab passwords are intentionally excluded from repository documentation.
