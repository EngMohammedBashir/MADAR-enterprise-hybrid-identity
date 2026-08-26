# 🏢 MADAR — Enterprise Identity & Workforce Access

## Phase 04 of the MADAR Cloud Transformation

> **Status: HYBRID IDENTITY VALIDATED — WORKSPACES END-TO-END PROOF COMPLETE / CLEANUP PENDING**  
> Corporate Active Directory → WireGuard hybrid connectivity → AWS Directory Service AD Connector → Amazon WorkSpaces → domain-user authentication → failure/recovery validation.

MADAR already migrated its representative legacy workload in Phase 03. Phase 04 addresses the next enterprise problem: **how employees use centralized corporate identity to access cloud-hosted workforce environments without creating a second disconnected identity store.**

The original plan included IAM Identity Center / SSO. During execution, that branch was intentionally not pursued because the lab was constrained to the AWS Free Plan and must not require an account-plan upgrade or Organizations changes simply to force an architectural diagram. The validated end-to-end consumer of the existing corporate directory became **Amazon WorkSpaces Personal**.

---

## 🎯 What this project proves

A synthetic MADAR employee defined only in the on-premises Active Directory can authenticate to an AWS-managed Windows desktop while the directory remains on-premises.

```text
👩 Sara Ibrahim
   sara.ibrahim@madar.local
          │
          │ corporate AD credentials
          ▼
☁️ Amazon WorkSpaces Personal
   WSAMZN-I0F8R2FL
   10.50.13.89
          │
          ▼
🔌 AWS Directory Service — AD Connector
   d-90667da553
          │
          ▼
🔐 Routed WireGuard hybrid network
          │
          ▼
🏢 MADAR-DC01
   192.168.14.10
   AD DS + DNS
   madar.local
```

The WorkSpace was not treated as "just another Windows VM." It was used as the final consumer that proved the entire hybrid identity chain.

---

## ✅ Current milestone

| Layer | Status | Validation |
|---|---|---|
| Active Directory / DNS | ✅ Complete | `madar.local` on `MADAR-DC01` |
| Workforce users/groups | ✅ Complete | departmental synthetic users + Global Security Groups |
| Domain client + GPO | ✅ Complete | `MADAR-CLIENT01` join, user login, GPO and firewall validation |
| Local least privilege | ✅ Complete | Sara allowed to IT share and denied Finance share |
| CGNAT-aware hybrid design | ✅ Complete | outbound-initiated self-managed WireGuard |
| AWS VPC/private routing | ✅ Complete | `10.50.0.0/16` with route to `192.168.14.0/24` |
| WireGuard tunnel | ✅ Complete | handshake, counters, routed traffic |
| AWS → AD protocols | ✅ Complete | DNS / Kerberos / LDAP / SMB diagnostics |
| AD Connector | ✅ Active | `d-90667da553` |
| WorkSpaces directory registration | ✅ Complete | `madar.local` registered to WorkSpaces |
| Dedicated WorkSpaces OU | ✅ Complete | `OU=WorkSpaces,OU=MADAR,DC=madar,DC=local` |
| WorkSpace domain join | ✅ Verified | `WSAMZN-I0F8R2FL.madar.local` created in on-prem AD |
| Domain-user cloud login | ✅ Verified | `madar\sara.ibrahim` authenticated to WorkSpaces |
| WorkSpace → on-prem DNS/DC path | ✅ Verified | source `10.50.13.89` reached `192.168.14.10:53` |
| VPN failure test | ✅ Verified | TCP/53 failed and DNS timed out after `wg0` shutdown |
| VPN recovery test | ✅ Verified | TCP/53 and DNS recovered after `wg0` restart |
| Final cost/resource cleanup | ⏳ Pending | stop/delete temporary cloud resources after documentation |

---

## 🧠 Why Client01 and WorkSpaces both exist

`MADAR-CLIENT01` and Amazon WorkSpaces look similar because both are Windows clients, but they prove different stages.

```text
MADAR-CLIENT01
  = local corporate endpoint
  = proves AD join, domain login, GPO and local authorization

Amazon WorkSpaces
  = AWS-managed cloud desktop
  = proves AWS can consume the on-prem directory end to end
```

Client01 proved the **directory itself** worked before AWS integration. WorkSpaces proved the **hybrid identity architecture** worked after AWS integration.

---

## 🌐 Implemented hybrid architecture

```text
HOME / VMware                                      AWS / us-east-1

MADAR-DC01                                         MADAR-P04-VPC
192.168.14.10                                      10.50.0.0/16
AD DS + DNS                                             |
      |                                                |
      v                                                v
MADAR-WG01   ===== encrypted WireGuard =====      MADAR-P04-WG-HUB
192.168.14.30      10.200.0.2 <-> 10.200.0.1      EC2 network appliance
                                                       |
                                                       +--> AD Connector
                                                       |    d-90667da553
                                                       |
                                                       +--> Amazon WorkSpaces
                                                            WSAMZN-I0F8R2FL
                                                            10.50.13.89
                                                            user: sara.ibrahim
```

The home lab sits behind Zain 5G carrier-grade NAT. A classic static Customer Gateway model was therefore not appropriate for this lab. `MADAR-WG01` initiates the WireGuard tunnel outbound toward a stable AWS-side EC2 endpoint.

> This project intentionally documents the VPN as **self-managed WireGuard on EC2**, not AWS managed Site-to-Site VPN.

---

## 🔌 AD Connector integration

The dedicated service account was created as:

```text
svc-adconnector
```

The account was verified as enabled, unlocked, non-expired and configured with a non-expiring lab password. No credential is stored in this repository.

The Connector initially failed in two different layers:

```text
Attempt 1
DNS unavailable / TCP 53
        ↓
network troubleshooting
        ↓
Attempt 2
Invalid credentials
        ↓
service-account validation/reset
        ↓
Fresh Connector
Stage = Active ✅
```

This is important because the failure evolution proves the troubleshooting process moved from networking to authentication rather than hiding all failures behind repeated resource recreation.

---

## 🖥️ Amazon WorkSpaces validation

The registered WorkSpaces directory uses:

```text
Directory ID : d-90667da553
Domain       : madar.local
Directory    : AD Connector
Registered   : True
Status       : Active
Target OU    : OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

A dedicated WorkSpaces OU was created and `svc-adconnector` was delegated the computer-object permissions required for the WorkSpaces domain join.

The test WorkSpace used:

```text
WorkSpace ID : ws-49q8s94dl
User         : sara.ibrahim
Computer     : WSAMZN-I0F8R2FL
Private IP   : 10.50.13.89
Bundle       : Standard Windows, Free tier eligible
Mode         : AutoStop after 1 hour
```

### Domain join proof

On `MADAR-DC01`:

```powershell
Get-ADComputer -Filter * `
  -SearchBase "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" `
  -Properties DNSHostName,Enabled |
Select-Object Name,DNSHostName,Enabled,DistinguishedName
```

Observed:

```text
Name            : WSAMZN-I0F8R2FL
DNSHostName     : WSAMZN-I0F8R2FL.madar.local
Enabled         : True
DistinguishedName:
CN=WSAMZN-I0F8R2FL,OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

### End-user authentication proof

Inside the WorkSpace:

```powershell
whoami
hostname
$env:USERDNSDOMAIN
ipconfig | findstr /i "IPv4 DNS"
```

Observed:

```text
madar\sara.ibrahim
WSAMZN-I0F8R2FL
MADAR.LOCAL
IPv4 Address : 10.50.13.89
```

That closes the end-to-end identity chain:

```text
On-prem AD user
      ↓
AD Connector
      ↓
Amazon WorkSpaces
      ↓
Domain joined Windows desktop
      ↓
Successful user authentication ✅
```

---

## 💥 Failure and recovery test

The strongest validation was not only a successful login. The routed identity dependency was intentionally broken and restored.

### Healthy baseline

From Sara's WorkSpace:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Observed:

```text
SourceAddress    : 10.50.13.89
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

### Injected failure

On `MADAR-WG01`:

```bash
sudo wg show
sudo wg-quick down wg0
```

The same WorkSpace tests then produced:

```text
TcpTestSucceeded : False
Resolve-DnsName  : timeout
```

### Recovery

The tunnel was restored:

```bash
sudo wg-quick up wg0
sudo wg show
```

The WorkSpace again returned:

```text
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

The final operational story is therefore:

```text
Healthy ✅
   ↓
WireGuard intentionally stopped 💥
   ↓
WorkSpace-to-AD connectivity lost ❌
   ↓
WireGuard restored 🔧
   ↓
TCP/DNS connectivity recovered ✅
```

---

## 🛠️ Important commands

### AWS subnet verification

```bash
AWS_PAGER="" aws ec2 describe-subnets \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,AvailabilityZoneId]' \
  --output table
```

### Hybrid route verification

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --route-table-ids rtb-08c2d8c0ea2bac825 \
  --query 'RouteTables[0].{Subnets:Associations[*].SubnetId,HybridRoute:Routes[?DestinationCidrBlock==`192.168.14.0/24`]}' \
  --output json
```

### AD Connector service-account state

```powershell
Get-ADUser svc-adconnector -Properties Enabled,LockedOut,PasswordExpired,PasswordNeverExpires |
Select-Object SamAccountName,Enabled,LockedOut,PasswordExpired,PasswordNeverExpires
```

### WorkSpaces OU computer query

```powershell
Get-ADComputer -Filter * `
  -SearchBase "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" `
  -Properties DNSHostName,Enabled |
Select-Object Name,DNSHostName,Enabled,DistinguishedName |
Format-Table -AutoSize
```

### WorkSpace hybrid baseline/recovery test

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

### WireGuard failure/recovery controls

```bash
sudo wg show
sudo wg-quick down wg0
sudo wg-quick up wg0
```

---

## 📸 Evidence highlights

### AD Connector active

![AD Connector Active](evidence/ad-connector-active-success.png)

### WireGuard tunnel

![WireGuard tunnel evidence](evidence/phase04-wireguard-tunnel-evidence.png)

### AWS-to-on-premises AD connectivity

![AWS to on-prem AD connectivity](evidence/phase04-aws-to-onprem-ad-connectivity-evidence.png)

### WorkSpace computer object created in on-prem AD

![WorkSpaces domain join verification](evidence/workspaces-onprem-ad-domain-join-verified.png)

### On-prem AD user authenticated to AWS WorkSpaces

![WorkSpaces hybrid AD authentication](evidence/workspaces-hybrid-ad-authentication-validation.png)

### Healthy WorkSpace-to-AD baseline

![WorkSpaces baseline connectivity](evidence/workspaces-to-onprem-baseline-connectivity.png)

### Injected VPN failure

![WorkSpaces VPN failure](evidence/workspaces-to-onprem-vpn-failure-test.png)

### VPN recovery validation

![WorkSpaces VPN recovery](evidence/workspaces-vpn-failure-recovery-validation.png)

### Local least-privilege proof

![Sara IT share access success](evidence/Sara-IT-Share-Access-Success.png)

![Sara Finance access denied](evidence/Sara-Finance-Access-Denied.png)

See [`evidence/README.md`](evidence/README.md) for the complete evidence map.

---

## 💰 Cost discipline

This account remained on the **AWS Free Plan**. The project explicitly avoided upgrading the account simply to enable a different identity architecture.

At the WorkSpaces launch checkpoint:

```text
AWS credit pool          : $180.00 issued
Estimated remaining      : $176.47
Estimated usage           : $3.53
Amazon WorkSpaces         : listed as an applicable credit product
Selected WorkSpaces bundle: Free tier eligible
```

The test WorkSpace used `AutoStop` and only one synthetic user. No NAT Gateway was created for this validation.

Temporary compute and local VMs are stopped when not being tested. Final deletion and billing review remain part of cleanup.

---

## ⚖️ Architecture decision: why IAM Identity Center was not forced

The initial architecture assumed:

```text
AD → AD Connector → IAM Identity Center → SSO/MFA → Permission Sets
```

That path was **not executed** in this Free Plan lab because it would have required account/organization changes that were outside the cost/account guardrails. Instead of upgrading the account merely to satisfy the initial diagram, the project preserved the validated corporate directory and proved real AWS consumption through WorkSpaces.

This is an engineering trade-off, not a hidden failure:

```text
Original goal      : centralized workforce SSO into AWS accounts
Lab constraint     : no Free Plan upgrade / no forced Organizations change
Validated outcome  : hybrid corporate AD consumed by AWS WorkSpaces
```

The IAM Identity Center architecture remains a future production extension for an account where the required organizational prerequisites are intentionally available.

---

## 📂 Repository structure

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

## 🚀 MADAR journey

```text
Phase 01  Cloud Foundation                     COMPLETE
Phase 02  Serverless Event Processing          COMPLETE
Phase 03  Legacy Migration & Data Center Exit  COMPLETE
Phase 04  Enterprise Identity & Workforce      HYBRID IDENTITY VALIDATED
          ├── Local AD / client                 COMPLETE
          ├── WireGuard hybrid network          COMPLETE
          ├── AD Connector                      ACTIVE
          ├── WorkSpaces domain join            VERIFIED
          ├── Domain-user cloud authentication  VERIFIED
          └── Failure / recovery                VERIFIED
Phase 05  Application Modernization            FUTURE
```

Master transformation record: `EngMohammedBashir/MADAR-cloud-transformation`.