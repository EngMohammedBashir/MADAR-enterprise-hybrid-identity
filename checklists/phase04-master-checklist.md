# Phase 04 Master Checklist

> **Checkpoint:** Hybrid identity is proven end to end through Amazon WorkSpaces. Documentation and final cleanup remain.

## A — Planning
- [x] business problem approved
- [x] current-state identity model documented
- [x] target hybrid identity architecture finalized for the lab
- [x] local workforce groups defined
- [x] AWS integration cost guardrail defined
- [x] cleanup/continuity strategy documented
- [x] record decision not to force IAM Identity Center / Organizations under Free Plan constraints

## B — Corporate directory lab
- [x] create `MADAR-DC01` in VMware
- [x] install Windows Server 2025
- [x] configure static IP `192.168.14.10`
- [x] install/configure AD DS and DNS
- [x] create `madar.local` forest/domain
- [x] create departmental OUs
- [x] create workforce Global Security Groups
- [x] create synthetic employee accounts
- [x] automate repeat user creation with PowerShell
- [x] verify group membership
- [x] capture directory baseline evidence

## B2 — Domain client / local authorization
- [x] create `MADAR-CLIENT01`
- [x] install Windows 11 Pro
- [x] point client DNS to `MADAR-DC01`
- [x] join client to `madar.local`
- [x] validate domain-user login
- [x] create/link `GPO-IT-Security`
- [x] verify GPO and Domain firewall enforcement
- [x] create departmental SMB shares
- [x] verify allowed IT-share access for `GG-IT`
- [x] verify denied Finance-share access for the IT user
- [x] capture positive/negative local authorization evidence

## C0 — Hybrid connectivity decision
- [x] target Region `us-east-1`
- [x] inspect AWS account / billing state
- [x] identify Zain 5G CGNAT constraint
- [x] reject classic static Customer Gateway design for this lab
- [x] select outbound-initiated self-managed WireGuard
- [x] document design honestly as EC2 WireGuard, not AWS managed Site-to-Site VPN

## C1 — Local WireGuard router
- [x] repurpose Ubuntu VM as `MADAR-WG01`
- [x] static IPv4 `192.168.14.30/24`
- [x] SSH connectivity validated
- [x] WireGuard installed
- [x] persistent IPv4 forwarding enabled
- [x] keypair created; private key protected
- [x] local reachability to `MADAR-DC01`
- [x] `madar.local` DNS resolution validated
- [x] configure local peer to AWS
- [x] configure `PersistentKeepalive = 25`
- [x] route AWS VPC CIDR through WireGuard `AllowedIPs`

## C2 — AWS network / WireGuard hub
- [x] create `MADAR-P04-VPC` with `10.50.0.0/16`
- [x] create public subnet for `WG-HUB`
- [x] create private subnets for Directory Service
- [x] launch Ubuntu EC2 `MADAR-P04-WG-HUB`
- [x] configure WireGuard
- [x] enable Linux IPv4 forwarding
- [x] disable source/destination check
- [x] configure Security Group for WireGuard and VPC transit traffic
- [x] configure forwarding/NAT rules
- [x] persist iptables rules
- [x] add private-subnet route `192.168.14.0/24` → `WG-HUB`
- [x] establish WireGuard handshake
- [x] verify transfer counters and routed traffic

## C3 — AWS → on-prem AD network validation
- [x] verify AWS-side reachability to `MADAR-DC01`
- [x] verify `madar.local` DNS resolution through `192.168.14.10`
- [x] verify TCP DNS 53
- [x] verify TCP Kerberos 88
- [x] verify TCP LDAP 389
- [x] verify TCP SMB 445
- [x] verify additional AD diagnostic ports
- [x] use packet capture to prove AWS-originated DNS traffic reaches DC01
- [x] prove DC01 reply returns to AWS

## C4 — AD Connector
- [x] create dedicated `svc-adconnector` account
- [x] validate account state: enabled / unlocked / password not expired
- [x] reset credential during troubleshooting without storing it in GitHub
- [x] capture initial `DNS unavailable` failure
- [x] fix route/transit/forwarding path
- [x] progress failure to `Invalid credentials`
- [x] create fresh AD Connector
- [x] wait for `Stage = Active`
- [x] capture AD Connector Active evidence
- [x] active Directory ID: `d-90667da553`

## D — Amazon WorkSpaces directory integration
- [x] inspect WorkSpaces directory registration state
- [x] confirm AD Connector is `Active`
- [x] verify WorkSpaces-compatible AZs
- [x] create `10.50.13.0/24` private subnet in `us-east-1c`
- [x] associate new subnet to the hybrid route table
- [x] register `madar.local` with Amazon WorkSpaces
- [x] verify directory `Registered = True / Active`
- [x] create `OU=WorkSpaces,OU=MADAR,DC=madar,DC=local`
- [x] delegate required computer-object permissions to `svc-adconnector`
- [x] configure WorkSpaces target OU

## E — WorkSpace end-to-end identity proof
- [x] verify `sara.ibrahim` is enabled in `madar.local`
- [x] select Standard Windows Free-tier-eligible bundle
- [x] select AutoStop after 1 hour
- [x] use one user only: `sara.ibrahim`
- [x] use 80 GB root / 10 GB user volume
- [x] create WorkSpace `ws-49q8s94dl`
- [x] verify computer `WSAMZN-I0F8R2FL`
- [x] verify private IP `10.50.13.89`
- [x] verify computer object exists in WorkSpaces OU
- [x] sign in to WorkSpace as `madar\sara.ibrahim`
- [x] verify `hostname = WSAMZN-I0F8R2FL`
- [x] verify `USERDNSDOMAIN = MADAR.LOCAL`
- [x] capture WorkSpaces authentication evidence

## F — Hybrid dependency failure / recovery
- [x] baseline WorkSpace → `192.168.14.10:53` succeeds
- [x] baseline `madar.local` DNS resolution succeeds
- [x] stop `wg0` with `sudo wg-quick down wg0`
- [x] verify TCP/53 fails from WorkSpace
- [x] verify DNS query times out
- [x] restore `wg0` with `sudo wg-quick up wg0`
- [x] verify TCP/53 succeeds again
- [x] verify DNS resolution succeeds again
- [x] capture baseline, failure and recovery screenshots

## G — IAM Identity Center original branch
- [x] execution-time decision recorded: do not force Organizations / account-plan changes for this Free Plan lab
- [ ] IAM Identity Center SSO — future production extension, not implemented
- [ ] MFA through Identity Center — future production extension
- [ ] permission sets / account assignments — future production extension
- [ ] AWS CLI SSO — future production extension
- [ ] CloudTrail SSO principal proof — future production extension

## H — Evidence / documentation
- [x] local AD / OU / users / groups evidence
- [x] domain client / login / GPO evidence
- [x] local allowed/denied authorization evidence
- [x] WireGuard evidence
- [x] AWS routing evidence
- [x] AWS-to-on-prem AD connectivity evidence
- [x] AD Connector Active screenshot
- [x] WorkSpaces domain-join screenshot
- [x] WorkSpaces domain-user authentication screenshot
- [x] WorkSpace healthy baseline screenshot
- [x] VPN failure screenshot
- [x] VPN recovery screenshot
- [x] README updated to final hybrid milestone
- [x] CURRENT-STATE updated
- [x] evidence index updated
- [ ] final resource/cost cleanup proof
- [ ] master MADAR transformation repository updated

## I — Pause / cleanup discipline
- [x] use AutoStop for the WorkSpace
- [x] stop temporary compute when tests are complete
- [x] power off local VMs when not required
- [ ] delete WorkSpace after documentation review
- [ ] deregister WorkSpaces directory when no longer needed
- [ ] delete AD Connector when no later phase requires it
- [ ] terminate `MADAR-P04-WG-HUB`
- [ ] release public IPv4 / Elastic IP if unused
- [ ] remove Phase 04-specific temporary routes/security rules if no continuity value
- [ ] final Bills / Credits / Cost Explorer review
- [ ] verify no paid Phase 04 resource remains unintentionally
- [ ] Phase 04 closeout decision recorded

---

## Resume checkpoint if another screenshot is required

```text
1. Power on MADAR-DC01
2. Power on MADAR-WG01
3. Start MADAR-P04-WG-HUB
4. Start WorkSpace if stopped
5. Verify WireGuard handshake
6. Verify WorkSpace -> 192.168.14.10 TCP/53
7. Verify Resolve-DnsName madar.local
```

**Do not rebuild the VPC, Active Directory, WireGuard tunnel or AD Connector unless a regression test proves that layer is broken.**