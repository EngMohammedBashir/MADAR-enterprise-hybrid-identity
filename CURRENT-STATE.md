# Phase 04 — Final State

**Status:** ✅ COMPLETED / VALIDATED / FAILURE-TESTED / CLEANED UP  
**Closeout date:** 2026-08-27  
**Primary theme:** Enterprise hybrid identity between on-premises Active Directory and AWS

## 🏁 Executive closeout

Phase 04 demonstrated a complete hybrid identity path from a VMware-hosted corporate Active Directory to an AWS-managed Amazon WorkSpaces desktop.

```text
sara.ibrahim
      ↓
Amazon WorkSpaces Personal
      ↓
AWS Directory Service — AD Connector
      ↓
AWS VPC private routing
      ↓
EC2 WireGuard routing appliance
      ↓
Encrypted WireGuard tunnel
      ↓
MADAR-WG01
      ↓
MADAR-DC01 / 192.168.14.10
      ↓
madar.local
```

The project did not stop at a healthy resource status. It proved domain join, corporate-user authentication, routed DNS/DC reachability, an intentional VPN outage, observed failure, successful recovery, and finally controlled cloud-resource cleanup.

## ✅ Validation gates

| Gate | Result |
|---|---|
| Active Directory / DNS baseline | ✅ Complete |
| Workforce users and security groups | ✅ Complete |
| Domain client / GPO / local authorization | ✅ Complete |
| CGNAT-aware hybrid design | ✅ Complete |
| WireGuard AWS ↔ on-premises connectivity | ✅ Complete |
| AWS → on-prem AD/DNS protocol path | ✅ Complete |
| AD Connector network/authentication | ✅ Complete |
| AD Connector Active | ✅ Complete |
| WorkSpaces directory registration | ✅ Complete |
| Dedicated WorkSpaces OU / delegated join | ✅ Complete |
| WorkSpace computer object in on-prem AD | ✅ Verified |
| `madar\sara.ibrahim` WorkSpaces login | ✅ Verified |
| WorkSpace → on-prem DNS/DC connectivity | ✅ Verified |
| VPN failure injection | ✅ Verified |
| VPN recovery | ✅ Verified |
| Evidence / documentation | ✅ Complete |
| AWS resource cleanup | ✅ Complete |
| Cost closeout | ✅ Reviewed |
| Phase 04 closeout | ✅ Accepted |

## 🧪 Key implementation evidence

Before cleanup, the validated cloud-side test resources included:

```text
AWS Region          : us-east-1
VPC                 : vpc-0371464657f10efb1 / 10.50.0.0/16
WG-HUB EC2          : i-029deb16c4c36fd11
AD Connector        : d-90667da553
WorkSpace           : ws-49q8s94dl
WorkSpace computer  : WSAMZN-I0F8R2FL
WorkSpace IP        : 10.50.13.89
WorkSpace user      : sara.ibrahim
Corporate DC/DNS    : MADAR-DC01 / 192.168.14.10
Domain              : madar.local
```

These identifiers are retained as historical lab evidence. The temporary AWS resources were subsequently removed.

## 💥 Failure / recovery result

Healthy baseline from the WorkSpace:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Observed healthy state:

```text
SourceAddress    : 10.50.13.89
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

Failure was intentionally injected on `MADAR-WG01`:

```bash
sudo wg-quick down wg0
```

The WorkSpace then observed:

```text
TcpTestSucceeded : False
Resolve-DnsName  : timeout
```

Recovery:

```bash
sudo wg-quick up wg0
sudo wg show
```

The same TCP/DNS tests returned to a successful state.

```text
Healthy ✅
   ↓
VPN intentionally disabled 💥
   ↓
WorkSpace loses on-prem AD/DNS reachability ❌
   ↓
VPN restored 🔧
   ↓
Connectivity recovered ✅
```

## 🧹 Final AWS cleanup

Cleanup was performed only after validation evidence had been captured.

Final verified state:

| Resource | Final state |
|---|---|
| Amazon WorkSpace `ws-49q8s94dl` | ✅ Deleted |
| WorkSpaces directory registration | ✅ Removed |
| AD Connector `d-90667da553` | ✅ Deleted |
| WG-HUB EC2 `i-029deb16c4c36fd11` | ✅ Terminated |
| Phase 04 Elastic IP | ✅ Released |
| WG-HUB security group | ✅ Deleted |
| Hybrid `192.168.14.0/24` route | ✅ Deleted |
| Phase 04 subnets | ✅ Deleted |
| Custom route tables | ✅ Deleted |
| Internet Gateway | ✅ Detached and deleted |
| VPC `vpc-0371464657f10efb1` | ✅ Deleted |

Final AWS CLI audit returned no WorkSpaces, no Directory Service directory, no Elastic IP and no Phase 04 VPC. The historical EC2 instance remained visible temporarily with state `terminated`, which is expected AWS behavior.

## 💰 Cost closeout

Cost Explorer was reviewed after cleanup.

```text
Gross month-to-date Usage/Fee : $ 2.1355 USD
AWS credits applied           : $-2.1355 USD
Calculated net                : $ 0.0000 USD
```

These values are a month-to-date account-level closeout snapshot, not a claim that every listed service charge originated only from Phase 04. The resource audit is the evidence that the temporary Phase 04 cloud infrastructure was removed.

Final closeout evidence:

![Phase 04 final closeout](evidence/phase04-final-closeout-evidence.png)

## ⚖️ IAM Identity Center decision

The original target architecture included IAM Identity Center, SSO/MFA and permission sets. That branch was not forced in this lab because the account was intentionally kept within the agreed AWS Free Plan/account guardrails and Organizations/account-plan changes were not introduced merely to satisfy the original diagram.

The proven Phase 04 scope is therefore:

```text
On-prem AD
   ↓
WireGuard hybrid network
   ↓
AD Connector
   ↓
Amazon WorkSpaces
   ↓
Domain-joined AWS-managed desktop
   ↓
Corporate domain-user authentication
   ↓
Failure / recovery validation
```

Direct AWS-account workforce SSO remains a future production extension when its organizational prerequisites are intentionally available.

## 📸 Evidence highlights

- `evidence/ad-connector-active-success.png`
- `evidence/phase04-wireguard-tunnel-evidence.png`
- `evidence/phase04-aws-to-onprem-ad-connectivity-evidence.png`
- `evidence/workspaces-onprem-ad-domain-join-verified.png`
- `evidence/workspaces-hybrid-ad-authentication-validation.png`
- `evidence/workspaces-to-onprem-baseline-connectivity.png`
- `evidence/workspaces-to-onprem-vpn-failure-test.png`
- `evidence/workspaces-vpn-failure-recovery-validation.png`
- `evidence/phase04-final-closeout-evidence.png`

## 🎯 Final outcome

Phase 04 is closed. No additional infrastructure is required to claim completion of the implemented hybrid-identity scope.

```text
PHASE 04
COMPLETED
   + VALIDATED
   + FAILURE-TESTED
   + RECOVERED
   + DOCUMENTED
   + CLEANED UP
   + COST-REVIEWED
```
