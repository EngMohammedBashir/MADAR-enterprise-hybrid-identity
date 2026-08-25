# Phase 04 Master Checklist

> **Checkpoint:** Hybrid network path is proven. Resume at AD Connector credential validation.

## A — Planning
- [x] business problem approved
- [x] current-state identity model documented
- [x] target AWS identity architecture finalized for the lab
- [x] local workforce groups defined
- [ ] AWS permission matrix finalized
- [ ] MFA/session strategy finalized for selected identity source
- [x] AWS integration cost guardrail defined
- [x] cleanup/continuity strategy documented

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
- [x] capture positive/negative authorization evidence

## C0 — Hybrid connectivity decision
- [x] target Region `us-east-1`
- [x] inspect AWS account / billing state
- [x] identify Zain 5G CGNAT constraint
- [x] verify local WAN address is private and public IPv4 is not stable
- [x] reject classic static Customer Gateway design for this lab
- [x] select outbound-initiated self-managed WireGuard
- [x] document design honestly as EC2 WireGuard, not AWS managed Site-to-Site VPN

## C1 — Local WireGuard router
- [x] repurpose retired Ubuntu VM as `MADAR-WG01`
- [x] hostname `madar-wg01`
- [x] static IPv4 `192.168.14.30/24`
- [x] default gateway `192.168.14.2`
- [x] SSH connectivity validated
- [x] WireGuard installed
- [x] persistent IPv4 forwarding enabled
- [x] keypair created; private key protected with mode `600`
- [x] local reachability to `MADAR-DC01`
- [x] `madar.local` DNS resolution validated
- [x] local TCP checks: 53 / 88 / 389 / 445
- [x] configure local WireGuard peer to AWS
- [x] configure `PersistentKeepalive = 25`
- [x] route AWS VPC CIDR through the tunnel using WireGuard `AllowedIPs`
- [x] retain Windows firewall rather than disabling it globally

## C2 — AWS network / WireGuard hub
- [x] create `MADAR-P04-VPC` with `10.50.0.0/16`
- [x] create public subnet for `WG-HUB`
- [x] create two private subnets in different AZs for AD Connector
- [x] attach/configure Internet Gateway and route tables
- [x] launch Ubuntu EC2 `MADAR-P04-WG-HUB`
- [x] assign stable AWS-side public endpoint for the lab
- [x] install/configure WireGuard on `WG-HUB`
- [x] enable Linux IPv4 forwarding
- [x] configure EC2 as a routing appliance
- [x] configure Security Group for WireGuard UDP/51820
- [x] allow required VPC transit traffic from `10.50.0.0/16` to the routing appliance
- [x] configure forwarding/NAT rules for AWS ↔ on-prem routed traffic
- [x] persist required iptables rules with `netfilter-persistent`
- [x] add private-subnet route `192.168.14.0/24` → `WG-HUB`
- [x] verify both private subnets use the intended route table
- [x] establish WireGuard handshake
- [x] verify tunnel ping `10.200.0.2` ↔ `10.200.0.1`
- [x] verify transfer counters increase in both directions

## C3 — AWS → on-prem AD network validation
- [x] verify AWS-side reachability to `MADAR-DC01` (`192.168.14.10`)
- [x] verify `madar.local` DNS resolution through `192.168.14.10`
- [x] verify TCP DNS 53
- [x] verify TCP Kerberos 88
- [x] verify TCP LDAP 389
- [x] verify TCP SMB 445
- [x] verify additional AD ports TCP 135 / 464 / 3268 during diagnostics
- [x] use packet capture to prove AWS-originated TCP/53 reaches DC01
- [x] prove DC01 TCP/53 reply returns to AWS
- [x] prove network failure is no longer the AD Connector blocker
- [ ] complete explicit UDP 53 / 88 / 389 validation if required by final acceptance evidence
- [ ] verify time/NTP suitability for Kerberos before final connector acceptance

## C4 — AD Connector
- [x] create dedicated `svc-adconnector` account
- [x] confirm service account exists and is enabled
- [x] point test Connector at `madar.local` / `192.168.14.10`
- [x] capture initial `DNS unavailable` failure
- [x] troubleshoot route tables / ENIs / Security Groups / packet path
- [x] correct AWS hub VPC-transit Security Group boundary
- [x] progress Connector failure from DNS connectivity to authentication
- [x] delete failed Connector at session close to control cost
- [ ] validate `svc-adconnector` account state: enabled / unlocked / password not expired
- [ ] reset credential to a known strong lab password if needed
- [ ] independently validate the credential before another Connector attempt
- [ ] re-check Directory Service pricing/free-trial eligibility
- [ ] create fresh AD Connector
- [ ] wait for `Stage = Active`
- [ ] capture AD Connector Active evidence

## D — IAM Identity Center foundation
- [ ] create/verify AWS Organizations state as required
- [ ] verify management-account ownership requirements
- [ ] enable/verify IAM Identity Center in `us-east-1`
- [ ] select the intended AD/Directory integration identity source
- [ ] verify intended users/groups are visible and usable

## E — Workforce authorization
- [ ] finalize AD-group → AWS-role mapping
- [ ] create Cloud Admin permission set
- [ ] create DevOps permission set
- [ ] create Developer permission set
- [ ] create Security permission set
- [ ] create Auditor permission set
- [ ] configure account assignments
- [ ] verify group-to-permission mappings

## F — Authentication / SSO
- [ ] validate AWS access portal
- [ ] validate SSO with a `madar.local` identity
- [ ] configure/validate MFA
- [ ] verify temporary sessions
- [ ] verify workforce users need no long-lived AWS access keys
- [ ] configure/test AWS CLI SSO

## G — Positive / negative AWS authorization tests
- [ ] Cloud Admin allowed action
- [ ] DevOps allowed action
- [ ] Developer allowed action
- [ ] Security read/investigation action
- [ ] Auditor read-only action
- [ ] Developer IAM-admin action denied
- [ ] Auditor write/delete denied
- [ ] Security infrastructure-admin action denied

## H — Identity lifecycle
- [ ] Joiner test
- [ ] Mover / group-role change test
- [ ] Leaver / disable test
- [ ] access revocation verified

## I — Audit
- [ ] CloudTrail Event History reviewed
- [ ] workforce principal/session identified
- [ ] allowed action evidence
- [ ] denied action evidence where available
- [ ] temporary-session evidence

## J — Evidence / documentation
- [x] local AD / OU / users / groups evidence
- [x] domain client / login / GPO evidence
- [x] local allowed/denied authorization evidence
- [x] local WireGuard readiness evidence
- [x] AWS VPC/subnet evidence
- [x] AWS route-table evidence
- [x] WireGuard tunnel evidence
- [x] AWS-to-on-prem AD connectivity evidence
- [x] README updated to current hybrid milestone
- [x] CURRENT-STATE updated to current hybrid milestone
- [x] evidence index updated
- [ ] AD Connector Active screenshot
- [ ] Identity Center screenshot
- [ ] permission-set / account-assignment screenshots
- [ ] SSO / MFA / CLI SSO proof
- [ ] AWS allowed/denied authorization proof
- [ ] lifecycle/offboarding proof
- [ ] CloudTrail audit proof
- [ ] final architecture diagram
- [ ] final cost/cleanup proof

## K — Pause / cleanup discipline
- [x] delete failed AD Connector before pausing
- [x] stop `MADAR-P04-WG-HUB` EC2 at session close
- [x] power off local VMs when not required
- [ ] release public IPv4 / Elastic IP when hybrid work is complete
- [ ] terminate temporary EC2 hub when no longer required
- [ ] remove Phase 04-specific routes/security rules when no longer required
- [ ] final Bills / Cost Explorer / resource audit
- [ ] verify no paid Phase 04 resource is unintentionally left running
- [ ] update master MADAR transformation repository
- [ ] Phase 04 closeout decision recorded

---

## Resume checkpoint

```text
1. Power on MADAR-DC01
2. Power on MADAR-WG01
3. Start MADAR-P04-WG-HUB
4. Verify WireGuard handshake
5. Verify DNS path to 192.168.14.10
6. Validate/reset svc-adconnector credential
7. Test credential
8. Create AD Connector
9. Target: Stage = Active
```

**Do not rebuild the VPC, WireGuard tunnel or routing path unless a regression test proves that layer is broken.**