# 🧭 MADAR Phase 04 — Full Lab Rebuild & Validation Runbook

> **Purpose:** Rebuild, validate, troubleshoot, failure-test and clean up the complete MADAR hybrid-identity lab without relying on chat history.
>
> **Scope:** VMware Active Directory → local WireGuard router → AWS VPC → EC2 WireGuard hub → AWS Directory Service AD Connector → Amazon WorkSpaces → domain-user authentication → VPN failure/recovery → cleanup.
>
> **Rule:** Never paste passwords, WireGuard private keys, AWS credentials, session tokens or MFA secrets into GitHub.

---

## 🗺️ 0. Final lab topology and known-good values

```text
HOME / VMware                                      AWS / us-east-1

MADAR-DC01                                         MADAR-P04-VPC
Windows Server 2025                                10.50.0.0/16
192.168.14.10                                           |
AD DS + DNS                                             |
madar.local                                              |
      |                                                  |
      v                                                  v
MADAR-WG01   ===== encrypted WireGuard =====      MADAR-P04-WG-HUB
Ubuntu              10.200.0.2 <-> 10.200.0.1     Ubuntu EC2 router
192.168.14.30                                      10.50.1.132
      |                                                  |
      |                                                  +--> AD Connector
      |                                                       d-90667da553
      |                                                  |
      |                                                  +--> WorkSpaces
      |                                                       ws-49q8s94dl
      |                                                       10.50.13.89
      |                                                       sara.ibrahim
      |                                                       WSAMZN-I0F8R2FL
      |
      +---------------- corporate identity ------------------+
```

### Known-good identifiers from the completed lab

| Item | Value |
|---|---|
| Region | `us-east-1` |
| Corporate domain | `madar.local` |
| Domain Controller / DNS | `192.168.14.10` |
| Local WG router | `192.168.14.30` |
| Local LAN | `192.168.14.0/24` |
| AWS VPC | `vpc-0371464657f10efb1` / `10.50.0.0/16` |
| Public WG subnet | `subnet-03db8481b69690b52` / `10.50.1.0/24` |
| Private subnet A | `subnet-0c6096cc338a611a1` / `10.50.11.0/24` / `us-east-1a` |
| Private subnet B | `subnet-00661aa39bb01f61a` / `10.50.12.0/24` / `us-east-1b` |
| WorkSpaces private subnet C | `subnet-05e3c3e6fea490ac1` / `10.50.13.0/24` / `us-east-1c` |
| Hybrid route table | `rtb-08c2d8c0ea2bac825` |
| AWS WG-HUB EC2 | `i-029deb16c4c36fd11` |
| WG-HUB ENI | `eni-019bce909122b9678` |
| WG-HUB private IP | `10.50.1.132` |
| WireGuard transit | `10.200.0.0/30` |
| AWS WG address | `10.200.0.1` |
| Local WG address | `10.200.0.2` |
| WireGuard interface | `wg0` |
| WireGuard AWS UDP port | `51820` |
| Local listening port observed | `48877` |
| AD Connector service account | `svc-adconnector` |
| AD Connector ID | `d-90667da553` |
| WorkSpaces OU | `OU=WorkSpaces,OU=MADAR,DC=madar,DC=local` |
| WorkSpace ID | `ws-49q8s94dl` |
| WorkSpace computer | `WSAMZN-I0F8R2FL` |
| WorkSpace IP | `10.50.13.89` |
| Test user | `sara.ibrahim` |

> IDs above document the completed run. A rebuild will normally create new IDs. Use names/CIDRs as the design reference and query the new IDs after creation.

---

# 🏢 1. On-premises Active Directory — baseline

## 1.1 Confirm DC identity

Run on `MADAR-DC01` PowerShell as Administrator:

```powershell
hostname
ipconfig
Get-ADDomain
Get-ADForest
```

### Why

This confirms that the machine you are about to trust as the corporate identity authority is actually the expected domain controller and that `madar.local` is healthy.

Think of the DC as the **company HR registry + phone book**. If its identity or DNS is wrong, every cloud integration above it will fail.

---

## 1.2 Verify AD users

```powershell
Get-ADUser -Filter * |
Select-Object SamAccountName,Name,Enabled,DistinguishedName |
Format-Table -AutoSize
```

Expected synthetic workforce:

```text
ahmed.ali
sara.ibrahim
omar.hassan
noura.saleh
khalid.mansour
svc-adconnector
```

To check only the custom Users OU:

```powershell
Get-ADUser -Filter * -SearchBase "OU=Users,OU=MADAR,DC=madar,DC=local" `
  -Properties Enabled,UserPrincipalName |
Select-Object SamAccountName,Name,Enabled,UserPrincipalName |
Format-Table -AutoSize
```

> A blank result here does **not** mean the users do not exist. In this lab the departmental users live in their departmental OUs, not necessarily under `OU=Users`.

---

## 1.3 Verify OU structure

```powershell
Get-ADOrganizationalUnit -Filter * |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

Expected custom structure:

```text
OU=MADAR,DC=madar,DC=local
├── Management
├── IT
├── Finance
├── HR
├── Sales
├── Users
├── Computers
├── Groups
└── WorkSpaces
```

---

## 1.4 Verify computers already in AD

```powershell
Get-ADComputer -Filter * |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

Before WorkSpaces, the expected notable objects included:

```text
MADAR-DC01
MADAR-CLIENT01
```

After WorkSpaces provisioning, expect an AWS-managed computer object such as `WSAMZN-I0F8R2FL`.

---

# 👥 2. Workforce groups and local authorization

## 2.1 Verify group membership

Example:

```powershell
Get-ADPrincipalGroupMembership sara.ibrahim |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

Expected: `GG-IT` plus normal domain groups.

Repeat for each synthetic user if rebuilding the full demonstration.

### Why

An OU answers **where an object is organized**. A security group answers **what access it receives**. Do not confuse the two.

---

## 2.2 Domain client verification

On `MADAR-CLIENT01`:

```powershell
whoami
hostname
ipconfig /all
```

Verify the client DNS points to `192.168.14.10`.

Test DNS:

```powershell
Resolve-DnsName madar.local -Server 192.168.14.10
```

Validate applied GPO:

```powershell
gpupdate /force
gpresult /r
```

Validate firewall profiles:

```powershell
Get-NetFirewallProfile |
Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction |
Format-Table -AutoSize
```

---

# 🔐 3. AD Connector service account

## 3.1 Confirm account health

Run on `MADAR-DC01`:

```powershell
Get-ADUser svc-adconnector -Properties Enabled,LockedOut,PasswordExpired,PasswordNeverExpires |
Select-Object SamAccountName,Enabled,LockedOut,PasswordExpired,PasswordNeverExpires
```

Known-good state:

```text
Enabled              : True
LockedOut            : False
PasswordExpired      : False
PasswordNeverExpires : True
```

### Why

If networking is green but this account is disabled/locked/expired, AD Connector will report an authentication failure. Treat credential failures as a separate layer from networking.

---

## 3.2 Check group membership

```powershell
Get-ADPrincipalGroupMembership svc-adconnector |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

The completed lab deliberately kept this account low privilege; it showed only normal `Domain Users` membership before OU delegation.

---

# 📁 4. Dedicated WorkSpaces OU and delegation

## 4.1 Create WorkSpaces OU if missing

```powershell
New-ADOrganizationalUnit `
  -Name "WorkSpaces" `
  -Path "OU=MADAR,DC=madar,DC=local" `
  -ProtectedFromAccidentalDeletion $true
```

Verify:

```powershell
Get-ADOrganizationalUnit -Filter 'Name -eq "WorkSpaces"' |
Select-Object Name,DistinguishedName,ProtectedFromAccidentalDeletion
```

Expected DN:

```text
OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

---

## 4.2 Inspect current ACL before delegation

```powershell
dsacls "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local"
```

Search specifically for the service account:

```powershell
dsacls "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" |
Select-String -Pattern "svc-adconnector" -Context 1,5
```

---

## 4.3 Delegate computer creation/control

The lab required `svc-adconnector` to create/manage WorkSpaces computer objects inside the dedicated OU.

A common `dsacls` mistake encountered was:

```text
computer is specified as Inherited Object Type. /I:S must be present.
```

That error means the inheritance scope was incomplete; do not assume delegation succeeded merely because the command was entered.

After correct delegation, validation showed entries equivalent to:

```text
Allow MADAR\svc-adconnector  SPECIAL ACCESS for computer
                              CREATE CHILD

Inherited to computer
Allow MADAR\svc-adconnector  FULL CONTROL
```

Always verify the resulting ACL with:

```powershell
dsacls "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" |
Select-String -Pattern "svc-adconnector" -Context 1,5
```

> The **verification output** is more important than memorizing one ACL command. Windows delegation syntax is easy to mistype; inspect the resulting ACL every time.

---

# 🌐 5. Local WireGuard gateway — `MADAR-WG01`

## 5.1 Verify host/network

```bash
hostname
ip addr
ip route
```

Known local gateway:

```text
MADAR-WG01
192.168.14.30/24
```

Verify DC reachability:

```bash
ping -c 4 192.168.14.10
```

Verify AD DNS from Linux:

```bash
nslookup madar.local 192.168.14.10
```

or:

```bash
dig @192.168.14.10 madar.local
```

---

## 5.2 Verify Linux forwarding

```bash
sysctl net.ipv4.ip_forward
```

Expected:

```text
net.ipv4.ip_forward = 1
```

Enable immediately if needed:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Persist it:

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-madar-forwarding.conf
sudo sysctl --system
```

---

## 5.3 WireGuard health

```bash
sudo wg show
```

Healthy output should show:

- interface `wg0`,
- peer present,
- endpoint pointing to the AWS public endpoint,
- AllowedIPs containing AWS VPC/tunnel ranges,
- recent handshake,
- non-zero transfer counters,
- persistent keepalive where configured.

Completed lab observation:

```text
interface: wg0
allowed ips: 10.200.0.1/32, 10.50.0.0/16
persistent keepalive: every 25 seconds
```

### Why

A handshake proves **the two VPN peers can talk**. It does **not** prove VPC workloads can traverse the EC2 router to Active Directory.

Think of it as two security guards confirming their radios work; that does not prove a delivery truck can pass every gate between buildings.

---

# ☁️ 6. AWS VPC/subnet discovery checks

Set pager off for cleaner CloudShell output:

```bash
export AWS_PAGER=""
```

or prefix commands with:

```bash
AWS_PAGER=""
```

---

## 6.1 List VPC subnets

```bash
AWS_PAGER="" aws ec2 describe-subnets \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,AvailabilityZoneId]' \
  --output table
```

Known completed topology:

```text
10.50.1.0/24   us-east-1a   public WG-HUB subnet
10.50.11.0/24  us-east-1a   private
10.50.12.0/24  us-east-1b   private
10.50.13.0/24  us-east-1c   WorkSpaces private subnet
```

---

## 6.2 Inspect specific private subnets

```bash
AWS_PAGER="" aws ec2 describe-subnets \
  --region us-east-1 \
  --subnet-ids \
    subnet-00661aa39bb01f61a \
    subnet-0c6096cc338a611a1 \
  --query 'Subnets[*].{SubnetId:SubnetId,CIDR:CidrBlock,AZ:AvailabilityZone,AZ_ID:AvailabilityZoneId,VpcId:VpcId,AvailableIPs:AvailableIpAddressCount,State:State,PublicIP:MapPublicIpOnLaunch}' \
  --output table
```

### Why

Directory/WorkSpaces integrations need compatible subnets and Availability Zones. Do not select two subnet IDs just because they are private; verify AZ placement and routing.

---

## 6.3 Check Availability Zone IDs

```bash
AWS_PAGER="" aws ec2 describe-availability-zones \
  --region us-east-1 \
  --query 'AvailabilityZones[?ZoneId==`use1-az2` || ZoneId==`use1-az4` || ZoneId==`use1-az6`].[ZoneName,ZoneId,State]' \
  --output table
```

This was used to understand which AZs were valid alternatives when WorkSpaces would not accept the original pair.

---

## 6.4 Create the additional WorkSpaces subnet

The completed lab created a third private subnet in `us-east-1c`:

```bash
AWS_PAGER="" aws ec2 create-subnet \
  --region us-east-1 \
  --vpc-id vpc-0371464657f10efb1 \
  --cidr-block 10.50.13.0/24 \
  --availability-zone us-east-1c \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=MADAR-P04-WorkSpaces-Private-C}]' \
  --query 'Subnet.{SubnetId:SubnetId,CIDR:CidrBlock,AZ:AvailabilityZone,AZ_ID:AvailabilityZoneId,State:State}' \
  --output table
```

Known created ID in the completed run:

```text
subnet-05e3c3e6fea490ac1
```

---

# 🛣️ 7. Route-table validation

## 7.1 Query private subnet route association

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --filters "Name=association.subnet-id,Values=subnet-00661aa39bb01f61a,subnet-0c6096cc338a611a1" \
  --query 'RouteTables[*].{RouteTableId:RouteTableId,Subnets:Associations[*].SubnetId,Routes:Routes[*].[DestinationCidrBlock,GatewayId,NatGatewayId,InstanceId,NetworkInterfaceId,State]}' \
  --output json
```

The critical route is:

```text
192.168.14.0/24 -> i-029deb16c4c36fd11 / eni-019bce909122b9678
```

---

## 7.2 Query all VPC route tables for the hybrid route

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'RouteTables[*].{RouteTableId:RouteTableId,Associations:Associations[*].SubnetId,HybridRoutes:Routes[?DestinationCidrBlock==`192.168.14.0/24`]}' \
  --output json
```

### Why

This answers two separate questions:

1. Does a route to on-prem exist?
2. Are the **actual subnets used by Directory Service / WorkSpaces** associated with the route table containing it?

A route in the wrong route table is functionally the same as no route.

---

## 7.3 Associate the additional subnet to the hybrid route table

```bash
AWS_PAGER="" aws ec2 associate-route-table \
  --region us-east-1 \
  --route-table-id rtb-08c2d8c0ea2bac825 \
  --subnet-id subnet-05e3c3e6fea490ac1 \
  --output json
```

Verify:

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --route-table-ids rtb-08c2d8c0ea2bac825 \
  --query 'RouteTables[0].{Subnets:Associations[*].SubnetId,HybridRoute:Routes[?DestinationCidrBlock==`192.168.14.0/24`]}' \
  --output json
```

---

# 🚦 8. AWS WG-HUB EC2 router checks

## 8.1 Inspect instance

```bash
AWS_PAGER="" aws ec2 describe-instances \
  --region us-east-1 \
  --instance-ids i-029deb16c4c36fd11 \
  --query 'Reservations[0].Instances[0].{InstanceId:InstanceId,State:State.Name,PrivateIP:PrivateIpAddress,PublicIP:PublicIpAddress,Subnet:SubnetId,VPC:VpcId}' \
  --output table
```

---

## 8.2 Source/destination check

Because this instance is a router, EC2 source/destination checking must be disabled.

Inspect:

```bash
AWS_PAGER="" aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id i-029deb16c4c36fd11 \
  --attribute sourceDestCheck
```

Disable if required:

```bash
AWS_PAGER="" aws ec2 modify-instance-attribute \
  --region us-east-1 \
  --instance-id i-029deb16c4c36fd11 \
  --no-source-dest-check
```

### Why

Normal EC2 instances are expected to send/receive traffic for themselves. A router must forward packets belonging to **other** hosts, so this guardrail must be disabled deliberately.

---

## 8.3 Linux routing checks on WG-HUB

```bash
ip addr
ip route
sudo wg show
sysctl net.ipv4.ip_forward
sudo iptables -S
sudo iptables -t nat -S
```

Key expectations:

```text
net.ipv4.ip_forward = 1
FORWARD rules permit the required VPC <-> on-prem path
NAT/MASQUERADE exists where required by the implemented topology
```

Persist iptables if using `netfilter-persistent`:

```bash
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

---

# 🛡️ 9. Security Group checks

## 9.1 WorkSpaces member SG

```bash
AWS_PAGER="" aws ec2 describe-security-groups \
  --region us-east-1 \
  --group-ids sg-06d324f74eea11f6b \
  --query 'SecurityGroups[0].{GroupId:GroupId,GroupName:GroupName,Description:Description,VpcId:VpcId,Ingress:IpPermissions,Egress:IpPermissionsEgress}' \
  --output json
```

Known member SG:

```text
d-90667da553_workspacesMembers
```

---

## 9.2 Directory controller SG

```bash
AWS_PAGER="" aws ec2 describe-security-groups \
  --region us-east-1 \
  --group-ids sg-0c7844aac31aa4737 \
  --query 'SecurityGroups[0].{GroupId:GroupId,GroupName:GroupName,Ingress:IpPermissions,Egress:IpPermissionsEgress}' \
  --output json
```

Known controller SG:

```text
d-90667da553_controllers
```

### Important troubleshooting lesson

The WG-HUB Security Group originally allowed WireGuard UDP but did not allow required VPC transit traffic. A healthy VPN handshake therefore coexisted with failed Directory Service DNS checks.

```text
WireGuard UDP allowed ✅
        does NOT imply
VPC transit traffic allowed ✅
```

---

# 🔎 10. Packet-level troubleshooting

When an AWS service says `DNS unavailable`, stop guessing. Use packet capture to find the boundary where the packet disappears.

On WG-HUB:

```bash
sudo tcpdump -ni ens5 host 192.168.14.10
```

In another terminal:

```bash
sudo tcpdump -ni wg0 host 192.168.14.10
```

For DNS specifically:

```bash
sudo tcpdump -ni ens5 host 192.168.14.10 and port 53
sudo tcpdump -ni wg0  host 192.168.14.10 and port 53
```

### Interpretation tree

```text
Packet absent on ens5
  -> AWS route / SG / subnet association / source side

Packet on ens5 but absent on wg0
  -> Linux forwarding / iptables / routing

Packet on wg0, no reply
  -> tunnel/local router/DC firewall/service/return path

Request + reply visible
  -> network is proven; investigate authentication/application layer
```

Observed Directory Service discovery included DNS queries such as:

```text
A? madar.local
SRV? _ldap._tcp.madar.local
SRV? _kerberos._tcp.madar.local
A? madar-dc01.madar.local
```

---

# 🔌 11. AWS Directory Service — AD Connector

## 11.1 Describe connector

```bash
AWS_PAGER="" aws ds describe-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'DirectoryDescriptions[0].{ID:DirectoryId,Name:Name,Type:Type,Stage:Stage}' \
  --output table
```

Healthy completed state was:

```text
Name  : madar.local
Type  : ADConnector
Stage : Active
```

---

## 11.2 Failure progression used for diagnosis

```text
DNS unavailable
   ↓ fix network path
Invalid credentials
   ↓ validate/reset svc-adconnector
Active ✅
```

Do **not** tear down a proven network just because the error changed to credentials. A changed error can be positive evidence that you successfully crossed a layer boundary.

---

# 🖥️ 12. Amazon WorkSpaces directory registration

## 12.1 Inspect registered WorkSpaces directory

```bash
AWS_PAGER="" aws workspaces describe-workspace-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'Directories[0].{DirectoryId:DirectoryId,State:State,Type:Type,SubnetIds:SubnetIds,WorkspaceSecurityGroupId:WorkspaceSecurityGroupId,Tenancy:Tenancy,SelfservicePermissions:SelfservicePermissions}' \
  --output json
```

Known registered state included:

```text
State                    : REGISTERED
WorkspaceSecurityGroupId : sg-06d324f74eea11f6b
Tenancy                  : SHARED
```

---

## 12.2 Locate Directory Service-created ENIs

```bash
AWS_PAGER="" aws ec2 describe-network-interfaces \
  --region us-east-1 \
  --filters "Name=description,Values=*d-90667da553*" \
  --query 'NetworkInterfaces[*].{ENI:NetworkInterfaceId,IP:PrivateIpAddress,Subnet:SubnetId,Description:Description,SGs:Groups[*].[GroupId,GroupName]}' \
  --output json
```

Completed run observed controller ENIs around:

```text
10.50.11.106
10.50.12.105
```

These helped prove which subnet/SG boundary Directory Service was actually using.

---

# 🧑‍💻 13. WorkSpace provisioning checks

## 13.1 List suitable bundles

```bash
AWS_PAGER="" aws workspaces describe-workspace-bundles \
  --region us-east-1 \
  --owner AMAZON \
  --query 'Bundles[*].{BundleId:BundleId,Name:Name,ComputeType:ComputeType.Name,RootGB:RootStorage.Capacity,UserGB:UserStorage.Capacity}' \
  --output table
```

Completed run selected the Free Tier eligible Standard Windows bundle:

```text
wsb-93xk71ss4
Standard with Windows 10 (Server 2022 based)
2 vCPU / 4 GiB
```

Use `AutoStop` for a short portfolio lab to reduce consumption.

---

## 13.2 Describe created WorkSpace

```bash
AWS_PAGER="" aws workspaces describe-workspaces \
  --region us-east-1 \
  --workspace-ids ws-49q8s94dl \
  --query 'Workspaces[0].{ID:WorkspaceId,User:UserName,State:State,IP:IpAddress,Directory:DirectoryId}' \
  --output table
```

Known-good result:

```text
Directory : d-90667da553
ID        : ws-49q8s94dl
IP        : 10.50.13.89
State     : AVAILABLE
User      : sara.ibrahim
```

---

# ✅ 14. Domain-join proof on the DC

After WorkSpaces creation, query the dedicated OU on `MADAR-DC01`:

```powershell
Get-ADComputer -Filter * `
  -SearchBase "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" `
  -Properties DNSHostName,Enabled |
Select-Object Name,DNSHostName,Enabled,DistinguishedName |
Format-Table -AutoSize
```

Expected proof:

```text
WSAMZN-I0F8R2FL
WSAMZN-I0F8R2FL.madar.local
Enabled = True
```

### Why this is strong evidence

The WorkSpaces console saying `AVAILABLE` proves the managed desktop exists. The computer object appearing **inside the on-prem AD OU** proves AWS successfully used the hybrid identity path to join the machine to the corporate domain.

---

# 🔑 15. End-user WorkSpace authentication proof

Inside Sara's WorkSpace PowerShell:

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

This closes the chain:

```text
Corporate AD identity
      ↓
AD Connector
      ↓
AWS-managed WorkSpace
      ↓
Domain joined computer
      ↓
Successful employee login ✅
```

---

# 🧪 16. WorkSpace → On-Prem healthy baseline

From the WorkSpace:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
```

Expected:

```text
SourceAddress       : 10.50.13.89
RemoteAddress       : 192.168.14.10
RemotePort          : 53
TcpTestSucceeded    : True
```

DNS test:

```powershell
Resolve-DnsName madar.local -Server 192.168.14.10
```

Expected:

```text
madar.local  A  192.168.14.10
```

### Why both tests matter

`Test-NetConnection` proves TCP reachability to port 53. `Resolve-DnsName` proves the service actually answers a DNS query. Port-open and application-response are related but not identical tests.

---

# 💥 17. Failure injection — intentionally cut WireGuard

## 17.1 Capture healthy VPN state first

On `MADAR-WG01`:

```bash
sudo wg show
```

Take evidence of a recent handshake and transfer counters.

---

## 17.2 Cut only the tunnel

```bash
sudo wg-quick down wg0
```

Do **not** power off the DC for this test. The point is to simulate **hybrid-path failure**, not server failure.

---

## 17.3 Re-run tests from WorkSpace

```powershell
Test-NetConnection 192.168.14.10 -Port 53
```

Expected failure:

```text
TcpTestSucceeded : False
```

Then:

```powershell
Resolve-DnsName madar.local -Server 192.168.14.10
```

Expected:

```text
operation timeout / DNS server unavailable
```

This is a successful **failure test** because the observed impact matches the designed dependency.

---

# ♻️ 18. Recovery validation

Bring the tunnel back:

```bash
sudo wg-quick up wg0
sudo wg show
```

Wait briefly for the handshake, then from the WorkSpace repeat:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

Expected recovery:

```text
TcpTestSucceeded : True
madar.local      : 192.168.14.10
```

### Failure/recovery mental model

```text
Healthy ✅
   ↓
wg0 DOWN 💥
   ↓
TCP/DNS LOST ❌
   ↓
wg0 UP 🔧
   ↓
handshake restored
   ↓
TCP/DNS RECOVERED ✅
```

---

# 🎛️ 19. Screenshot-friendly validation dashboard

Run as one PowerShell block inside the WorkSpace:

```powershell
& {
    Clear-Host

    $DC = "192.168.14.10"
    $Domain = "madar.local"

    Write-Host ""
    Write-Host "============================================================================" -ForegroundColor Cyan
    Write-Host "              MADAR HYBRID CLOUD - VPN RECOVERY VALIDATION" -ForegroundColor Cyan
    Write-Host "============================================================================" -ForegroundColor Cyan
    Write-Host ""

    Write-Host "  FAILURE SCENARIO EXECUTED" -ForegroundColor Yellow
    Write-Host "  ------------------------------------------------------------------------" -ForegroundColor DarkGray
    Write-Host "  Test                 : WireGuard tunnel interruption" -ForegroundColor White
    Write-Host "  Observed Impact      : WorkSpace lost connectivity to On-Prem DC" -ForegroundColor White
    Write-Host "  Failure Evidence     : TCP/53 failed + DNS query timed out" -ForegroundColor Red

    Write-Host ""
    Write-Host "  LIVE RECOVERY TEST" -ForegroundColor Yellow
    Write-Host "  ------------------------------------------------------------------------" -ForegroundColor DarkGray

    $Tcp = Test-NetConnection $DC -Port 53 -WarningAction SilentlyContinue

    Write-Host "  Source WorkSpace IP  : " -NoNewline -ForegroundColor Gray
    Write-Host $Tcp.SourceAddress -ForegroundColor Cyan

    Write-Host "  Target DC            : " -NoNewline -ForegroundColor Gray
    Write-Host "$DC`:53" -ForegroundColor Cyan

    Write-Host "  TCP Connectivity     : " -NoNewline -ForegroundColor Gray
    if ($Tcp.TcpTestSucceeded) { Write-Host "PASS" -ForegroundColor Green }
    else { Write-Host "FAIL" -ForegroundColor Red }

    try {
        $Dns = Resolve-DnsName $Domain -Server $DC -ErrorAction Stop |
               Where-Object { $_.Type -eq "A" } |
               Select-Object -First 1

        Write-Host "  DNS Resolution       : " -NoNewline -ForegroundColor Gray
        Write-Host "PASS" -ForegroundColor Green
        Write-Host "  $Domain      : " -NoNewline -ForegroundColor Gray
        Write-Host $Dns.IPAddress -ForegroundColor Green
        $DnsPass = $true
    }
    catch {
        Write-Host "  DNS Resolution       : " -NoNewline -ForegroundColor Gray
        Write-Host "FAIL" -ForegroundColor Red
        $DnsPass = $false
    }

    Write-Host ""
    Write-Host "  FAILURE -> RECOVERY RESULT" -ForegroundColor Yellow
    Write-Host "  ------------------------------------------------------------------------" -ForegroundColor DarkGray
    Write-Host "  [OBSERVED] VPN tunnel down  -> TCP/DNS connectivity LOST" -ForegroundColor Red

    if ($Tcp.TcpTestSucceeded -and $DnsPass) {
        Write-Host "  [VERIFIED] VPN restored     -> TCP/DNS connectivity RECOVERED" -ForegroundColor Green
        Write-Host ""
        Write-Host "============================================================================" -ForegroundColor Cyan
        Write-Host "                     STATUS: RECOVERY SUCCESSFUL" -ForegroundColor Green
        Write-Host "============================================================================" -ForegroundColor Cyan
    }
    else {
        Write-Host "  [FAILED] Connectivity has not fully recovered" -ForegroundColor Red
    }

    Write-Host ""
}
```

> The failure line is a summary of the separately captured failure evidence. The recovery checks are live. Retain the raw `TcpTestSucceeded : False` / DNS timeout screenshot as primary failure evidence.

---

# 🎨 20. WorkSpaces identity validation dashboard

Run as one block to avoid PowerShell prompts breaking the screenshot:

```powershell
& {
    Clear-Host

    $ComputerName = $env:COMPUTERNAME
    $User         = (whoami)
    $Domain       = $env:USERDNSDOMAIN
    $PrivateIP    = "10.50.13.89"
    $WorkspaceID  = "ws-49q8s94dl"

    Write-Host ""
    Write-Host "=========================================================================" -ForegroundColor Cyan
    Write-Host "                 MADAR HYBRID CLOUD - WORKSPACES" -ForegroundColor Cyan
    Write-Host "                     VALIDATION REPORT" -ForegroundColor DarkCyan
    Write-Host "=========================================================================" -ForegroundColor Cyan
    Write-Host ""

    Write-Host "  IDENTITY & AUTHENTICATION" -ForegroundColor Yellow
    Write-Host "  ---------------------------------------------------------------------" -ForegroundColor DarkGray
    Write-Host "  Logged-in User     : " -NoNewline -ForegroundColor Gray
    Write-Host $User -ForegroundColor Green
    Write-Host "  Active Directory   : " -NoNewline -ForegroundColor Gray
    Write-Host $Domain -ForegroundColor Green

    Write-Host ""
    Write-Host "  AWS WORKSPACE" -ForegroundColor Yellow
    Write-Host "  ---------------------------------------------------------------------" -ForegroundColor DarkGray
    Write-Host "  Computer Name      : " -NoNewline -ForegroundColor Gray
    Write-Host $ComputerName -ForegroundColor Cyan
    Write-Host "  Private IP         : " -NoNewline -ForegroundColor Gray
    Write-Host $PrivateIP -ForegroundColor Cyan
    Write-Host "  WorkSpace ID       : " -NoNewline -ForegroundColor Gray
    Write-Host $WorkspaceID -ForegroundColor Cyan

    Write-Host ""
    Write-Host "  HYBRID CLOUD VALIDATION" -ForegroundColor Yellow
    Write-Host "  ---------------------------------------------------------------------" -ForegroundColor DarkGray
    Write-Host "  [PASS] " -NoNewline -ForegroundColor Green
    Write-Host "On-Premises AD authentication successful" -ForegroundColor White
    Write-Host "  [PASS] " -NoNewline -ForegroundColor Green
    Write-Host "WorkSpace joined to MADAR.LOCAL" -ForegroundColor White
    Write-Host "  [PASS] " -NoNewline -ForegroundColor Green
    Write-Host "Hybrid identity path operational" -ForegroundColor White

    Write-Host ""
    Write-Host "=========================================================================" -ForegroundColor Cyan
    Write-Host "                    STATUS: VALIDATION SUCCESSFUL" -ForegroundColor Green
    Write-Host "=========================================================================" -ForegroundColor Cyan
    Write-Host ""
}
```

---

# 💰 21. Billing / Free Plan checks

Free-tier API query attempted during the lab:

```bash
AWS_PAGER="" aws freetier get-free-tier-usage \
  --region us-east-1 \
  --filter '{
    "Dimensions": {
      "Key": "SERVICE",
      "Values": ["Amazon WorkSpaces"],
      "MatchOptions": ["EQUALS"]
    }
  }' \
  --output json
```

An empty result does not by itself prove the service is free. Always verify the Billing/Free Tier/Credits console and current product eligibility before provisioning.

The account used for this lab had promotional Free Plan credits and the selected Standard WorkSpaces bundle was explicitly shown by the console as **Free tier eligible**.

---

# 🧹 22. Cleanup — WorkSpace

## 22.1 Verify exact target first

```bash
AWS_PAGER="" aws workspaces describe-workspaces \
  --region us-east-1 \
  --workspace-ids ws-49q8s94dl \
  --query 'Workspaces[0].{ID:WorkspaceId,User:UserName,State:State,IP:IpAddress,Directory:DirectoryId}' \
  --output table
```

### Why

Treat destructive cleanup like decommissioning a production asset: identify it first, then delete it.

---

## 22.2 Terminate WorkSpace

Correct CLI structure:

```bash
AWS_PAGER="" aws workspaces terminate-workspaces \
  --region us-east-1 \
  --terminate-workspace-requests '[{"WorkspaceId":"ws-49q8s94dl"}]' \
  --output json
```

Successful request:

```json
{
  "FailedRequests": []
}
```

### Important syntax trap

This form was rejected:

```bash
--terminate-workspace-requests WorkspaceId=ws-49q8s94dl
```

because AWS interpreted the literal string incorrectly for this list structure.

---

## 22.3 Poll termination

```bash
AWS_PAGER="" aws workspaces describe-workspaces \
  --region us-east-1 \
  --workspace-ids ws-49q8s94dl \
  --query 'Workspaces[0].{ID:WorkspaceId,State:State,User:UserName}' \
  --output table
```

Expected progression:

```text
AVAILABLE -> TERMINATING -> no result
```

---

# 🧹 23. Cleanup — deregister WorkSpaces directory

Attempt only after all WorkSpaces are gone:

```bash
AWS_PAGER="" aws workspaces deregister-workspace-directory \
  --region us-east-1 \
  --directory-id d-90667da553
```

If you receive:

```text
There are WorkSpaces assigned to this directory.
```

wait until WorkSpace termination completes and retry.

Verify no registration remains:

```bash
AWS_PAGER="" aws workspaces describe-workspace-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'Directories[0].{DirectoryId:DirectoryId,State:State}' \
  --output table
```

No returned row means the directory is no longer registered with WorkSpaces.

---

# 🧹 24. Cleanup — AD Connector

Inspect first:

```bash
AWS_PAGER="" aws ds describe-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'DirectoryDescriptions[0].{ID:DirectoryId,Name:Name,Type:Type,Stage:Stage}' \
  --output table
```

After WorkSpaces deregistration the connector may temporarily appear `Inoperable`; that does not mean you should leave it running indefinitely.

Delete:

```bash
AWS_PAGER="" aws ds delete-directory \
  --region us-east-1 \
  --directory-id d-90667da553 \
  --output json
```

Poll:

```bash
AWS_PAGER="" aws ds describe-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'DirectoryDescriptions[0].{ID:DirectoryId,Stage:Stage}' \
  --output table
```

Expected:

```text
Deleting -> no result
```

---

# 🧹 25. Cleanup — WG-HUB Elastic IP and EC2

## 25.1 Verify EC2 target

```bash
AWS_PAGER="" aws ec2 describe-instances \
  --region us-east-1 \
  --instance-ids i-029deb16c4c36fd11 \
  --query 'Reservations[0].Instances[0].{InstanceId:InstanceId,State:State.Name,PrivateIP:PrivateIpAddress,PublicIP:PublicIpAddress,Subnet:SubnetId,VPC:VpcId}' \
  --output table
```

---

## 25.2 Find attached Elastic IP

```bash
AWS_PAGER="" aws ec2 describe-addresses \
  --region us-east-1 \
  --filters "Name=instance-id,Values=i-029deb16c4c36fd11" \
  --query 'Addresses[*].{AllocationId:AllocationId,PublicIp:PublicIp,AssociationId:AssociationId,NetworkInterfaceId:NetworkInterfaceId}' \
  --output table
```

Completed run values:

```text
AllocationId  : eipalloc-0774bdb03a3d7e1fd
AssociationId : eipassoc-09f1489da20709371
PublicIp      : 34.228.95.241
ENI           : eni-019bce909122b9678
```

---

## 25.3 Disassociate and release EIP

```bash
AWS_PAGER="" aws ec2 disassociate-address \
  --region us-east-1 \
  --association-id eipassoc-09f1489da20709371
```

```bash
AWS_PAGER="" aws ec2 release-address \
  --region us-east-1 \
  --allocation-id eipalloc-0774bdb03a3d7e1fd
```

---

## 25.4 Terminate WG-HUB

```bash
AWS_PAGER="" aws ec2 terminate-instances \
  --region us-east-1 \
  --instance-ids i-029deb16c4c36fd11 \
  --query 'TerminatingInstances[0].{InstanceId:InstanceId,Previous:PreviousState.Name,Current:CurrentState.Name}' \
  --output table
```

Expected:

```text
running -> shutting-down -> terminated
```

---

# 🧽 26. Final network-resource audit

Do not blindly delete the VPC until dependencies are enumerated.

List subnets:

```bash
AWS_PAGER="" aws ec2 describe-subnets \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,State]' \
  --output table
```

List route tables:

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'RouteTables[*].{RouteTableId:RouteTableId,Associations:Associations[*].SubnetId,Routes:Routes[*].[DestinationCidrBlock,GatewayId,InstanceId,NetworkInterfaceId,State]}' \
  --output json
```

List SGs:

```bash
AWS_PAGER="" aws ec2 describe-security-groups \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'SecurityGroups[*].[GroupId,GroupName,Description]' \
  --output table
```

List ENIs:

```bash
AWS_PAGER="" aws ec2 describe-network-interfaces \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'NetworkInterfaces[*].[NetworkInterfaceId,Status,PrivateIpAddress,SubnetId,Description]' \
  --output table
```

List EIPs:

```bash
AWS_PAGER="" aws ec2 describe-addresses \
  --region us-east-1 \
  --query 'Addresses[*].[AllocationId,PublicIp,AssociationId,InstanceId,NetworkInterfaceId]' \
  --output table
```

### Cleanup rule

```text
Discover dependencies
      ↓
Identify Phase-04-only resources
      ↓
Delete child dependencies first
      ↓
Delete route/SG/subnet resources
      ↓
Delete IGW/VPC only when empty
      ↓
Check Billing/Cost Explorer
```

---

# 🔧 27. AWS CLI output hygiene / common shell mistakes

## Disable the pager

AWS CLI may open `less`, which caused confusing output during the lab.

Use:

```bash
export AWS_PAGER=""
```

or per command:

```bash
AWS_PAGER="" aws ...
```

If already inside `less`, press:

```text
q
```

### Do not paste shell prompt text

Never include:

```text
~ $
```

when pasting commands. It is the shell prompt, not part of the command.

A malformed paste such as repeated `~ $` can produce unrelated Bash errors like trying to execute the home directory.

---

# 🧠 28. Troubleshooting decision tree

```text
AD Connector / WorkSpaces problem
        |
        +--> Does on-prem DC/DNS work locally?
        |       └── NO -> fix AD/DNS first
        |
        +--> Is WireGuard handshake recent?
        |       └── NO -> fix peer/endpoint/CGNAT path
        |
        +--> Do AWS subnets have 192.168.14.0/24 route?
        |       └── NO -> fix route table/association
        |
        +--> Does packet reach WG-HUB ens5?
        |       └── NO -> AWS route/SG/source boundary
        |
        +--> Does packet leave wg0?
        |       └── NO -> Linux forwarding/iptables/routing
        |
        +--> Does DC reply?
        |       └── NO -> local route/firewall/service
        |
        +--> DNS works but connector says credentials?
        |       └── validate svc-adconnector account/delegation
        |
        +--> Connector Active but WorkSpace join fails?
                └── inspect WorkSpaces OU + delegation + subnet availability
```

---

# 📸 29. Evidence checklist

Capture proof at these gates:

```text
🏢 DC/domain verification
👥 users + security groups
💻 Client01 domain join
🛡️ GPO/firewall enforcement
✅ IT share allowed
🚫 Finance share denied
🔐 WireGuard handshake
🛣️ AWS hybrid route
📡 AWS -> on-prem AD connectivity
🔌 AD Connector Active
📁 WorkSpaces OU delegation
☁️ WorkSpaces registration
🖥️ WorkSpace AVAILABLE
🔗 WorkSpace computer object in on-prem AD
👩 madar\sara.ibrahim logged in
✅ WorkSpace -> DC baseline
💥 VPN failure
♻️ VPN recovery
💰 final cleanup/billing
```

Recommended filenames used in this lab:

```text
workspaces-hybrid-ad-authentication-validation.png
workspaces-to-onprem-baseline-connectivity.png
workspaces-to-onprem-vpn-failure-test.png
workspaces-vpn-failure-recovery-validation.png
```

---

# 🏁 30. Acceptance criteria

The lab is genuinely complete only when the engineer can explain and prove each layer:

```text
AD identity exists                              ✅
AD identity has meaningful group authorization ✅
Local domain client works                       ✅
Hybrid VPN peers establish                      ✅
AWS workload traffic traverses the VPN          ✅
AD DNS/Kerberos/LDAP path is usable             ✅
AD Connector authenticates                      ✅
WorkSpaces consumes the corporate directory     ✅
AWS cloud desktop joins on-prem domain           ✅
Domain user signs in                             ✅
VPN failure causes expected service impact       ✅
VPN restoration recovers connectivity            ✅
Temporary cloud resources are cleaned up         ✅
```

## Final engineering lesson

```text
Handshake       != routed connectivity
Ping            != application readiness
Open TCP port   != correct DNS/authentication
AD Connector    != WorkSpace domain join
AVAILABLE       != user authentication proof
Successful test != resilient system
Cleanup         != optional portfolio housekeeping
```

The strength of this project is the **layer-by-layer proof** and the ability to reproduce the reasoning, not merely the number of AWS services created.
