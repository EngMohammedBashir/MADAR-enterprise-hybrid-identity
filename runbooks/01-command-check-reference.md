# ⚡ MADAR Phase 04 — Command & Health-Check Reference

> Fast reference after you already understand the architecture. For the full rebuild, explanations, expected outputs and troubleshooting logic, use [`00-lab-rebuild-and-validation.md`](00-lab-rebuild-and-validation.md).

---

## 🏢 Active Directory

```powershell
hostname
ipconfig
Get-ADDomain
Get-ADForest
```

```powershell
Get-ADUser -Filter * |
Select-Object SamAccountName,Name,Enabled,DistinguishedName |
Format-Table -AutoSize
```

```powershell
Get-ADOrganizationalUnit -Filter * |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

```powershell
Get-ADComputer -Filter * |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

```powershell
Get-ADPrincipalGroupMembership sara.ibrahim |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

```powershell
Get-ADUser svc-adconnector -Properties Enabled,LockedOut,PasswordExpired,PasswordNeverExpires |
Select-Object SamAccountName,Enabled,LockedOut,PasswordExpired,PasswordNeverExpires
```

```powershell
Get-ADPrincipalGroupMembership svc-adconnector |
Select-Object Name,DistinguishedName |
Format-Table -AutoSize
```

---

## 📁 WorkSpaces OU / ACL

```powershell
Get-ADOrganizationalUnit -Filter 'Name -eq "WorkSpaces"' |
Select-Object Name,DistinguishedName,ProtectedFromAccidentalDeletion
```

```powershell
dsacls "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local"
```

```powershell
dsacls "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" |
Select-String -Pattern "svc-adconnector" -Context 1,5
```

```powershell
Get-ADComputer -Filter * `
  -SearchBase "OU=WorkSpaces,OU=MADAR,DC=madar,DC=local" `
  -Properties DNSHostName,Enabled |
Select-Object Name,DNSHostName,Enabled,DistinguishedName |
Format-Table -AutoSize
```

---

## 💻 Domain client

```powershell
whoami
hostname
ipconfig /all
Resolve-DnsName madar.local -Server 192.168.14.10
gpupdate /force
gpresult /r
```

```powershell
Get-NetFirewallProfile |
Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction |
Format-Table -AutoSize
```

---

## 🔐 Local WireGuard gateway

```bash
hostname
ip addr
ip route
ping -c 4 192.168.14.10
nslookup madar.local 192.168.14.10
sysctl net.ipv4.ip_forward
sudo wg show
```

Failure injection:

```bash
sudo wg-quick down wg0
```

Recovery:

```bash
sudo wg-quick up wg0
sudo wg show
```

---

## ☁️ AWS CLI basics

```bash
export AWS_PAGER=""
```

```bash
aws sts get-caller-identity
aws configure get region
```

---

## 🌐 Subnets

```bash
AWS_PAGER="" aws ec2 describe-subnets \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,AvailabilityZoneId]' \
  --output table
```

```bash
AWS_PAGER="" aws ec2 describe-availability-zones \
  --region us-east-1 \
  --query 'AvailabilityZones[*].[ZoneName,ZoneId,State]' \
  --output table
```

---

## 🛣️ Routes

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'RouteTables[*].{RouteTableId:RouteTableId,Associations:Associations[*].SubnetId,HybridRoutes:Routes[?DestinationCidrBlock==`192.168.14.0/24`]}' \
  --output json
```

```bash
AWS_PAGER="" aws ec2 describe-route-tables \
  --region us-east-1 \
  --route-table-ids rtb-08c2d8c0ea2bac825 \
  --query 'RouteTables[0].{Subnets:Associations[*].SubnetId,HybridRoute:Routes[?DestinationCidrBlock==`192.168.14.0/24`]}' \
  --output json
```

---

## 🚦 WG-HUB EC2

```bash
AWS_PAGER="" aws ec2 describe-instances \
  --region us-east-1 \
  --instance-ids i-029deb16c4c36fd11 \
  --query 'Reservations[0].Instances[0].{InstanceId:InstanceId,State:State.Name,PrivateIP:PrivateIpAddress,PublicIP:PublicIpAddress,Subnet:SubnetId,VPC:VpcId}' \
  --output table
```

```bash
AWS_PAGER="" aws ec2 describe-instance-attribute \
  --region us-east-1 \
  --instance-id i-029deb16c4c36fd11 \
  --attribute sourceDestCheck
```

On the instance:

```bash
ip addr
ip route
sudo wg show
sysctl net.ipv4.ip_forward
sudo iptables -S
sudo iptables -t nat -S
```

Packet capture:

```bash
sudo tcpdump -ni ens5 host 192.168.14.10
sudo tcpdump -ni wg0 host 192.168.14.10
sudo tcpdump -ni ens5 host 192.168.14.10 and port 53
sudo tcpdump -ni wg0 host 192.168.14.10 and port 53
```

---

## 🛡️ Security Groups

```bash
AWS_PAGER="" aws ec2 describe-security-groups \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'SecurityGroups[*].[GroupId,GroupName,Description]' \
  --output table
```

---

## 🔌 AD Connector

```bash
AWS_PAGER="" aws ds describe-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'DirectoryDescriptions[0].{ID:DirectoryId,Name:Name,Type:Type,Stage:Stage}' \
  --output table
```

---

## 🖥️ WorkSpaces directory

```bash
AWS_PAGER="" aws workspaces describe-workspace-directories \
  --region us-east-1 \
  --directory-ids d-90667da553 \
  --query 'Directories[0].{DirectoryId:DirectoryId,State:State,SubnetIds:SubnetIds,WorkspaceSecurityGroupId:WorkspaceSecurityGroupId,Tenancy:Tenancy}' \
  --output json
```

```bash
AWS_PAGER="" aws ec2 describe-network-interfaces \
  --region us-east-1 \
  --filters "Name=description,Values=*d-90667da553*" \
  --query 'NetworkInterfaces[*].{ENI:NetworkInterfaceId,IP:PrivateIpAddress,Subnet:SubnetId,Description:Description,SGs:Groups[*].[GroupId,GroupName]}' \
  --output json
```

---

## 🧑‍💻 WorkSpace

```bash
AWS_PAGER="" aws workspaces describe-workspaces \
  --region us-east-1 \
  --workspace-ids ws-49q8s94dl \
  --query 'Workspaces[0].{ID:WorkspaceId,User:UserName,State:State,IP:IpAddress,Directory:DirectoryId}' \
  --output table
```

Inside WorkSpace:

```powershell
whoami
hostname
$env:USERDNSDOMAIN
ipconfig | findstr /i "IPv4 DNS"
```

Hybrid checks:

```powershell
Test-NetConnection 192.168.14.10 -Port 53
Resolve-DnsName madar.local -Server 192.168.14.10
```

---

## 🧹 Cleanup

Terminate WorkSpace:

```bash
AWS_PAGER="" aws workspaces terminate-workspaces \
  --region us-east-1 \
  --terminate-workspace-requests '[{"WorkspaceId":"ws-49q8s94dl"}]' \
  --output json
```

Deregister WorkSpaces directory:

```bash
AWS_PAGER="" aws workspaces deregister-workspace-directory \
  --region us-east-1 \
  --directory-id d-90667da553
```

Delete AD Connector:

```bash
AWS_PAGER="" aws ds delete-directory \
  --region us-east-1 \
  --directory-id d-90667da553 \
  --output json
```

Find EIP:

```bash
AWS_PAGER="" aws ec2 describe-addresses \
  --region us-east-1 \
  --filters "Name=instance-id,Values=i-029deb16c4c36fd11" \
  --query 'Addresses[*].{AllocationId:AllocationId,PublicIp:PublicIp,AssociationId:AssociationId,NetworkInterfaceId:NetworkInterfaceId}' \
  --output table
```

Disassociate/release:

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

Terminate WG-HUB:

```bash
AWS_PAGER="" aws ec2 terminate-instances \
  --region us-east-1 \
  --instance-ids i-029deb16c4c36fd11 \
  --query 'TerminatingInstances[0].{InstanceId:InstanceId,Previous:PreviousState.Name,Current:CurrentState.Name}' \
  --output table
```

Final dependency scans:

```bash
AWS_PAGER="" aws ec2 describe-network-interfaces \
  --region us-east-1 \
  --filters "Name=vpc-id,Values=vpc-0371464657f10efb1" \
  --query 'NetworkInterfaces[*].[NetworkInterfaceId,Status,PrivateIpAddress,SubnetId,Description]' \
  --output table
```

```bash
AWS_PAGER="" aws ec2 describe-addresses \
  --region us-east-1 \
  --query 'Addresses[*].[AllocationId,PublicIp,AssociationId,InstanceId,NetworkInterfaceId]' \
  --output table
```

---

## 🧠 Fast fault-isolation rule

```text
AD local check
   ↓
WireGuard handshake
   ↓
AWS route association
   ↓
WG-HUB SG
   ↓
ens5 packet capture
   ↓
Linux forwarding/NAT
   ↓
wg0 packet capture
   ↓
DC DNS/AD response
   ↓
service-account credentials/delegation
   ↓
WorkSpaces registration/domain join
   ↓
end-user authentication
```
