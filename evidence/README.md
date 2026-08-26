# 📸 Phase 04 Evidence Index

This folder contains the proof behind the Phase 04 engineering claims.

> Evidence is treated as proof, not decoration. Each screenshot should answer one engineering question.

## 1 — Local identity / Active Directory

| Evidence file | What it proves |
|---|---|
| `legacy-vm-running-before-identity-modernization.png` | representative legacy environment remained available before identity modernization |
| `Domain-Controller-Verification.png` | `MADAR-DC01`, `madar.local` and domain-controller role verification |
| `Active-Directory-Domain-Controller.png` | domain controller visible in Active Directory |
| `Active-Directory-OU-Structure.png` | departmental OU structure |
| `Active-Directory-Security-Groups.png` | workforce Global Security Groups |
| `Manual-AD-User-Creation.png` | first synthetic user created manually |
| `PowerShell-Automated-AD-Users-Creation.png` | repeatable user creation through PowerShell |
| `AD-Security-Group-Membership-Verification.png` | synthetic users mapped to intended groups |
| `GPO-IT-Domain-Firewall-Policy.png` | IT Domain firewall policy configuration |
| `CLIENT01-System-Verification.png` | Windows client baseline |
| `CLIENT01-Domain-Join-Success.png` | `MADAR-CLIENT01` joined `madar.local` |
| `Domain-User-Login-Sara.png` | local domain-user authentication |
| `GPO-IT-Security-Applied.png` | IT security GPO applied |
| `Domain-Firewall-GPO-Verification.png` | firewall enforcement verified |
| `Sara-IT-Share-Access-Success.png` | intended IT-share access succeeded |
| `Sara-Finance-Access-Denied.png` | cross-department Finance-share access denied |

## 2 — Hybrid connectivity

| Evidence file | What it proves |
|---|---|
| `MADAR-WG01-Local-PreAWS-Validation.png` | local Linux gateway readiness |
| `AWS-VPC-Subnets-Validation.png` | Phase 04 VPC/subnet foundation before cleanup |
| `AWS-Route-Tables-Validation.png` | AWS routing used by the hybrid path |
| `phase04-wireguard-tunnel-evidence.png` | encrypted WireGuard peer relationship and traffic |
| `phase04-aws-to-onprem-ad-connectivity-evidence.png` | AWS-side path reached on-prem AD/DNS |

A handshake alone proves only peer connectivity. The routed application path was validated separately with AWS-originated AD/DNS traffic.

## 3 — AWS Directory Service

| Evidence file | What it proves |
|---|---|
| `ad-connector-active-success.png` | AD Connector for `madar.local` reached `Active` |

Troubleshooting progressed through distinct layers:

```text
DNS unavailable
      ↓
fix transit / routing / forwarding
      ↓
Invalid credentials
      ↓
validate/reset svc-adconnector
      ↓
AD Connector Active ✅
```

## 4 — Amazon WorkSpaces end-to-end identity

| Evidence file | What it proves |
|---|---|
| `workspaces-onprem-ad-domain-join-verified.png` | AWS WorkSpace computer object exists in the on-prem WorkSpaces OU |
| `workspaces-hybrid-ad-authentication-validation.png` | `madar\sara.ibrahim` authenticated inside the AWS WorkSpace |
| `workspaces-to-onprem-baseline-connectivity.png` | WorkSpace `10.50.13.89` reached `MADAR-DC01` TCP/53 and resolved `madar.local` |
| `workspaces-to-onprem-vpn-failure-test.png` | stopping WireGuard caused expected TCP/DNS failure |
| `workspaces-vpn-failure-recovery-validation.png` | restoring WireGuard restored WorkSpace-to-AD connectivity |

```text
On-prem AD user
      ↓
AD Connector
      ↓
WorkSpace provisioned
      ↓
Computer object created in on-prem AD
      ↓
Corporate user login succeeds
      ↓
Healthy DNS/DC path verified
      ↓
VPN intentionally stopped
      ↓
Failure observed
      ↓
VPN restored
      ↓
Recovery verified
```

## 5 — Final cleanup and cost closeout

| Evidence file | What it proves |
|---|---|
| `phase04-final-closeout-evidence.png` | final live AWS audit showed WorkSpaces deleted, AD Connector deleted, WG-HUB terminated, Elastic IP released and Phase 04 VPC deleted; Cost Explorer closeout was also reviewed |

![Phase 04 final closeout evidence](phase04-final-closeout-evidence.png)

At the captured closeout checkpoint:

```text
Amazon WorkSpaces          DELETED
AD Connector               DELETED
WireGuard EC2 Hub          TERMINATED
Elastic IP                 RELEASED
Phase 04 VPC               DELETED

Gross month-to-date usage  $ 2.1355 USD
AWS credits applied        $-2.1355 USD
Calculated net             $ 0.0000 USD
```

The cost values are an account-level month-to-date snapshot. They are not attributed exclusively to Phase 04. Resource-specific cleanup is proven by the AWS inventory checks shown in the same closeout evidence.

## 6 — What is intentionally not claimed

IAM Identity Center SSO/MFA, permission sets and CLI SSO were not implemented in this Free Plan lab. Organizations/account-plan changes were not forced solely to satisfy the initial design.

Therefore this evidence folder intentionally does **not** claim proof for:

- IAM Identity Center SSO,
- Identity Center MFA,
- permission-set assignments,
- AWS CLI SSO,
- CloudTrail SSO-principal activity.

Those remain future production-extension targets.

## 🔐 Evidence rules

- Never expose passwords, MFA seeds, private keys, reusable access keys or session tokens.
- Never commit WireGuard private keys.
- Prefer one strong screenshot per engineering claim over repetitive screenshots.
- Keep intentional failures when they prove a dependency or least-privilege boundary.
- Crop or compose evidence so a reviewer can understand the claim quickly.
- Lab passwords are intentionally excluded from repository documentation.
