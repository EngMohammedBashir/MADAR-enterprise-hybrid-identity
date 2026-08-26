# Phase 04 — Hybrid Connectivity Troubleshooting Case Study

## Why this document exists

This is the troubleshooting record for the hardest networking and identity checkpoints in Phase 04.

The goal is not to preserve every command. The goal is to preserve the **reasoning model** that turned vague AWS Directory Service failures into specific, testable diagnoses and then proved the dependency with a controlled outage.

## Failure chain

The project encountered three meaningful layers:

```text
1. AD Connector -> DNS unavailable
2. AD Connector -> Invalid credentials
3. WorkSpaces dependency -> intentional VPN outage
```

Each was diagnosed separately.

## Incident 1 — AD Connector reported DNS unavailable

Symptom:

```text
DNS unavailable (TCP port 53) for IP 192.168.14.10
```

Possible causes included DNS service, Windows firewall, VPC route, subnet association, EC2 network-appliance configuration, WireGuard AllowedIPs, Linux forwarding, Security Groups, NACLs or return path.

The packet was treated like a parcel moving through checkpoints:

```text
AD Connector ENI
      ↓
Private subnet
      ↓
Route table
      ↓
EC2 Security Group
      ↓
WG-HUB ens5
      ↓
Linux forwarding / NAT
      ↓
WG-HUB wg0
      ↓
WireGuard tunnel
      ↓
MADAR-WG01
      ↓
MADAR-DC01:53
      ↓
Return path
```

### Key discovery

The EC2 `WG-HUB` Security Group originally admitted WireGuard UDP/51820 but did not admit the VPC transit traffic required when the EC2 instance acted as a router.

After correcting the VPC transit boundary and verifying Linux forwarding/NAT, `tcpdump` showed AWS-originated TCP/53 traffic reach `192.168.14.10` and receive replies.

Lesson:

> A healthy WireGuard handshake proves peer connectivity, not routed workload connectivity.

## Incident 2 — failure moved to credentials

Once networking worked, the Connector failure changed to:

```text
Invalid credentials (bad username/password)
```

That was useful evidence. It showed the network barrier had been removed and the integration reached the authentication layer.

The dedicated account was checked:

```powershell
Get-ADUser svc-adconnector -Properties Enabled,LockedOut,PasswordExpired,PasswordNeverExpires |
Select-Object SamAccountName,Enabled,LockedOut,PasswordExpired,PasswordNeverExpires
```

Validated state:

```text
Enabled              : True
LockedOut            : False
PasswordExpired      : False
PasswordNeverExpires : True
```

After credential reset/validation, a fresh Connector reached:

```text
Stage = Active
Directory ID = d-90667da553
```

## WorkSpaces integration lesson

After the Connector became Active, Amazon WorkSpaces was used as a real directory consumer.

WorkSpaces registration required compatible AZs. The existing `us-east-1a` subnet was not offered for the second WorkSpaces subnet choice, so a new private subnet was created in `us-east-1c` and associated with the existing hybrid route table.

This preserved the identity path without redesigning the working tunnel.

## Domain join proof

A dedicated OU was created:

```text
OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

The WorkSpace computer appeared there as:

```text
WSAMZN-I0F8R2FL.madar.local
```

That proved AWS successfully performed the domain-join operation against the on-premises AD.

## End-user authentication proof

Inside the AWS WorkSpace:

```powershell
whoami
hostname
$env:USERDNSDOMAIN
```

Observed:

```text
madar\sara.ibrahim
WSAMZN-I0F8R2FL
MADAR.LOCAL
```

This moved the project from "the Connector is green" to **real domain-user authentication in an AWS-managed desktop**.

## Incident 3 — controlled WireGuard outage

A healthy baseline was first recorded from the WorkSpace:

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

Then the tunnel was intentionally disabled on `MADAR-WG01`:

```bash
sudo wg-quick down wg0
```

The same WorkSpace tests produced:

```text
TcpTestSucceeded : False
Resolve-DnsName  : timeout
```

The tunnel was restored:

```bash
sudo wg-quick up wg0
```

The same tests returned to success.

## Why the failure test matters

A successful screenshot can sometimes hide an accidental alternative route. The controlled outage proved the WorkSpace genuinely depended on the designed hybrid path.

```text
Healthy tunnel
      ↓
WorkSpace reaches on-prem DC
      ↓
Tunnel removed
      ↓
Connectivity fails
      ↓
Tunnel restored
      ↓
Connectivity recovers
```

That is stronger architectural evidence than a one-time ping.

## Diagnostic method retained

```text
Observe
  ↓
Form one hypothesis
  ↓
Test one boundary
  ↓
Change one thing
  ↓
Re-test
```

Avoid changing route + firewall + WireGuard + credentials + directory settings simultaneously. A portfolio project is stronger when the engineer can explain **which layer failed, why it failed, and which evidence proved the fix**.

## Final lessons

```text
WireGuard handshake     != routed application connectivity
Ping                    != DNS/Kerberos/LDAP readiness
Route table entry       != packet reached EC2
DNS connectivity        != valid AD credentials
AD Connector Active     != end-user authentication
WorkSpace READY         != hybrid dependency proven
Failure + recovery test  = dependency proven operationally
```

Each layer requires its own proof.
