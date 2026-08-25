# Phase 04 Master Checklist

## A — Planning
- [x] business problem approved
- [x] current-state identity model documented
- [x] target AWS identity architecture finalized for the lab
- [x] local workforce groups defined
- [ ] AWS permission matrix finalized
- [ ] MFA/session strategy finalized for selected identity source
- [x] exact AWS integration cost guardrail defined for the lab
- [x] cleanup/continuity strategy documented

## B — Corporate directory lab
- [x] create `MADAR-DC01` in VMware
- [x] install Windows Server 2025
- [x] configure static IP for the domain controller
- [x] install/configure AD DS
- [x] configure DNS
- [x] create `madar.local` forest/domain
- [x] create organizational units
- [x] create workforce security groups
- [x] create synthetic employee accounts
- [x] automate repeat user creation with PowerShell
- [x] verify group membership
- [x] capture directory baseline evidence

## B2 — Domain client and local authorization validation
- [x] create `MADAR-CLIENT01`
- [x] install Windows 11 Pro
- [x] point client DNS to `MADAR-DC01`
- [x] validate DNS connectivity to `madar.local`
- [x] join `MADAR-CLIENT01` to `madar.local`
- [x] validate domain-user login
- [x] create and link `GPO-IT-Security`
- [x] verify GPO application on the client
- [x] verify Domain firewall policy
- [x] create departmental SMB shares for authorization testing
- [x] verify allowed IT-share access for `GG-IT`
- [x] verify denied Finance-share access for the IT user
- [x] capture positive and negative local authorization evidence

## C0 — Hybrid connectivity decision / preflight
- [x] confirm AWS Region target: `us-east-1`
- [x] confirm AWS account identity
- [x] confirm account is not currently in AWS Organizations
- [x] confirm IAM Identity Center is not currently enabled
- [x] inspect current AWS credits / billing state before new resources
- [x] identify local WAN environment as Zain 5G behind CGNAT
- [x] verify router WAN address is private (`10.x.x.x`)
- [x] verify public IPv4 changes after router reconnect/restart
- [x] reject classic static-IP AWS Site-to-Site VPN path for this lab
- [x] select self-managed routed WireGuard tunnel as the lab connectivity path
- [x] document why the selected design is a CGNAT-aware lab workaround rather than a claim that it is AWS managed Site-to-Site VPN

## C1 — Local WireGuard routing layer
- [x] repurpose the retired `MADAR-LEGACY01` VM as `MADAR-WG01`
- [x] rename Ubuntu hostname to `madar-wg01`
- [x] keep `MADAR-WG01` on the same VMware network as `MADAR-DC01`
- [x] verify VMware DHCP pool before assigning a static address
- [x] assign static IPv4 `192.168.14.30/24`
- [x] verify default gateway `192.168.14.2`
- [x] verify SSH connectivity on the static address
- [x] install WireGuard / wireguard-tools
- [x] enable persistent IPv4 forwarding
- [x] create WireGuard keypair locally
- [x] protect the private key with mode `600`
- [x] verify `MADAR-WG01` can reach `MADAR-DC01` on `192.168.14.10`
- [x] verify `madar.local` resolves through `MADAR-DC01`
- [x] verify TCP connectivity from WG01 to DC01 on DNS 53
- [x] verify TCP connectivity from WG01 to DC01 on Kerberos 88
- [x] verify TCP connectivity from WG01 to DC01 on LDAP 389
- [x] verify TCP connectivity from WG01 to DC01 on SMB 445
- [ ] configure the local WireGuard peer to initiate outbound to AWS
- [ ] configure `PersistentKeepalive = 25` for CGNAT resilience
- [ ] add the AWS VPC CIDR route on `MADAR-DC01` via `MADAR-WG01`
- [ ] restrict Windows Firewall rules to required AD/DNS traffic instead of disabling the firewall globally

## C2 — AWS network and WireGuard hub
- [ ] create Phase 04 VPC with a non-overlapping CIDR
- [ ] create one public subnet for the EC2 WireGuard hub
- [ ] create two private subnets in different Availability Zones for AD Connector
- [ ] create/associate Internet Gateway for the public subnet
- [ ] configure public and private route tables intentionally
- [ ] launch a small Ubuntu EC2 instance as `WG-HUB`
- [ ] assign a public IPv4 / Elastic IP as required for the lab
- [ ] install WireGuard on `WG-HUB`
- [ ] enable IPv4 forwarding on `WG-HUB`
- [ ] disable EC2 source/destination check
- [ ] restrict the EC2 Security Group to the required WireGuard and VPC traffic
- [ ] add private-subnet route: `192.168.14.0/24` -> `WG-HUB` ENI / appliance target
- [ ] establish the WireGuard handshake between `MADAR-WG01` and `WG-HUB`
- [ ] verify bidirectional routed connectivity between the AWS VPC and `192.168.14.0/24`

## C3 — AD network validation before AD Connector
- [ ] verify AWS-side reachability to `MADAR-DC01` (`192.168.14.10`)
- [ ] verify DNS connectivity to the domain controller on TCP/UDP 53
- [ ] verify Kerberos connectivity on TCP/UDP 88
- [ ] verify LDAP connectivity on TCP/UDP 389
- [ ] verify time/NTP is suitable for Kerberos
- [ ] verify DNS resolution for `madar.local` from the AWS-side test path
- [ ] do not create AD Connector until the above tests pass

## C4 — AWS identity foundation
- [ ] create/verify AWS Organizations state as required for the selected IAM Identity Center deployment
- [ ] verify management-account ownership requirements for the selected AD integration path
- [ ] enable/verify IAM Identity Center in `us-east-1`
- [ ] re-check current Directory Service pricing/free-trial eligibility immediately before creation
- [ ] create AD Connector only after network validation
- [ ] point AD Connector at `madar.local` / `192.168.14.10`
- [ ] wait for AD Connector status to become Active and troubleshoot only from validated routing/DNS/AD facts
- [ ] select AD Connector / Active Directory as the IAM Identity Center identity source
- [ ] verify intended users/groups are visible/usable through the selected integration

## D — Workforce authorization
- [ ] finalize mapping from local AD groups to AWS workforce roles
- [ ] create Cloud Admin permission set
- [ ] create DevOps permission set
- [ ] create Developer permission set
- [ ] create Security permission set
- [ ] create Auditor permission set
- [ ] configure account assignments
- [ ] verify group-to-permission mappings

## E — Authentication
- [ ] validate AWS access portal
- [ ] validate SSO login using a `madar.local` identity
- [ ] configure/validate MFA
- [ ] verify temporary sessions
- [ ] verify no employee long-lived AWS access keys are required
- [ ] configure/test AWS CLI SSO

## F — Positive AWS access tests
- [ ] Cloud Admin allowed action
- [ ] DevOps allowed action
- [ ] Developer allowed action
- [ ] Security allowed read/investigation action
- [ ] Auditor read-only action

## G — Negative AWS access tests
- [ ] Developer IAM-admin action denied
- [ ] Auditor write/delete denied
- [ ] Security infrastructure-admin action denied
- [ ] role boundary failure captured as expected evidence

## H — Identity lifecycle
- [ ] onboarding test
- [ ] group/role-change test
- [ ] offboarding/disable test
- [ ] access revocation verified

## I — Audit
- [ ] CloudTrail Event History reviewed
- [ ] workforce principal/session identified in audit data
- [ ] allowed action evidence captured
- [ ] denied action evidence captured where available
- [ ] temporary session evidence captured

## J — Evidence and documentation
- [x] domain-controller verification
- [x] OU structure evidence
- [x] security-group evidence
- [x] manual user-creation evidence
- [x] PowerShell user-automation evidence
- [x] group-membership verification
- [x] Windows client baseline evidence
- [x] domain-join evidence
- [x] domain-user login evidence
- [x] GPO configuration/application evidence
- [x] local allowed/denied authorization evidence
- [ ] CGNAT/public-IP preflight evidence
- [ ] local WireGuard gateway readiness evidence (target: one consolidated terminal screenshot)
- [ ] WireGuard architecture/ADR
- [ ] WireGuard handshake / routed-connectivity evidence
- [ ] AD protocol validation evidence before connector creation
- [ ] final AWS architecture diagram
- [ ] AD Connector Active screenshot
- [ ] Identity Center screenshot
- [ ] permission-set screenshot
- [ ] account-assignment screenshot
- [ ] SSO login screenshot
- [ ] MFA proof
- [ ] CLI SSO proof
- [ ] AWS negative-test proof
- [ ] offboarding proof
- [ ] audit proof
- [ ] final cost/cleanup proof

### Screenshot discipline from C1 onward
- [ ] C1 local gateway: one consolidated terminal evidence screenshot
- [ ] C2 AWS hub: one console/network screenshot plus one WireGuard handshake screenshot
- [ ] C3 AD network validation: one consolidated AWS-side protocol test screenshot
- [ ] C4 identity foundation: connector Active + Identity Center source screenshot(s)
- [ ] D/E/F/G: capture only screenshots proving permission mapping, login/MFA, and intentional allow/deny boundaries
- [ ] H/I: lifecycle revocation and CloudTrail evidence
- [ ] K: final zero-running-cost / cleanup evidence

## K — Cleanup / continuity
- [ ] delete AD Connector when no longer required
- [ ] terminate the temporary EC2 WireGuard hub when no longer required
- [ ] release public IPv4 / Elastic IP resources that should not remain allocated
- [ ] remove Phase 04-specific routes/security rules that should not persist
- [ ] power off `MADAR-WG01` and local identity VMs when not required
- [x] retain local identity VMs powered off when not required
- [ ] preserve workforce identity configuration for Phase 05+ only if cost-safe and intentionally required
- [ ] final AWS Cost Explorer / Bills / resource audit
- [ ] verify no paid Phase 04 resource is unintentionally left running
- [ ] update implementation repository README and CURRENT-STATE
- [ ] update master transformation repository
- [ ] Phase 04 closeout decision recorded
