# Phase 04 Target and Validated Identity Architecture

## Validated lab architecture

The architecture that was actually implemented and proven is:

```text
MADAR Corporate Workforce
          |
          v
Representative Microsoft Active Directory
MADAR-DC01 / 192.168.14.10
madar.local
          |
          v
MADAR-WG01 / WireGuard
192.168.14.30
          |
          | encrypted routed tunnel
          v
AWS EC2 WireGuard routing appliance
MADAR-P04-WG-HUB
          |
          v
AWS Directory Service AD Connector
 d-90667da553
          |
          v
Amazon WorkSpaces Personal
WSAMZN-I0F8R2FL / 10.50.13.89
          |
          v
sara.ibrahim authenticates with madar.local credentials
```

## Why the architecture changed

The initial target assumed IAM Identity Center, SSO/MFA and permission sets. Execution-time account constraints changed the lab path.

The account was intentionally kept on the AWS Free Plan. The project guardrail prohibited upgrading the account or changing Organizations state merely to force the original architecture. Rather than document an unimplemented SSO design as success, the project selected an AWS service that directly consumes the AD Connector and could prove the hybrid identity path end to end: Amazon WorkSpaces Personal.

## Design rules

1. Keep the corporate identity source authoritative in `madar.local`.
2. Do not expose `MADAR-DC01` directly to the public Internet.
3. Treat the EC2 WireGuard host as a routing appliance, not an ordinary endpoint.
4. Use a dedicated service account (`svc-adconnector`) for directory integration.
5. Place WorkSpaces computer objects in a dedicated OU.
6. Validate behavior, not only green console status.
7. Intentionally test failure and recovery of the hybrid dependency.
8. Keep the AWS account on its intended Free Plan; do not upgrade merely for a portfolio lab.
9. Stop/delete temporary paid resources after evidence is captured.

## WorkSpaces directory placement

A dedicated OU was created:

```text
OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

`svc-adconnector` received the required computer-object delegation in that OU. The WorkSpaces directory was configured to target it.

## Network path

```text
WorkSpace 10.50.13.89
        ↓
AWS VPC 10.50.0.0/16
        ↓
route: 192.168.14.0/24 -> WG-HUB
        ↓
WireGuard 10.200.0.0/30
        ↓
MADAR-WG01 192.168.14.30
        ↓
MADAR-DC01 192.168.14.10
```

## Future production extension

For a production AWS organization where IAM Identity Center organizational prerequisites are intentionally enabled, the corporate identity model can be extended toward:

```text
Corporate identity
      ↓
IAM Identity Center
      ↓
MFA + SSO
      ↓
Group-based permission sets
      ↓
Temporary AWS sessions
```

That future extension is architectural direction only; it is not claimed as executed in this lab.
