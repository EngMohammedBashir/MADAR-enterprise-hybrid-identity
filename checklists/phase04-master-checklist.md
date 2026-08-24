# Phase 04 Master Checklist

## A — Planning
- [x] business problem approved
- [x] current-state identity model documented
- [ ] target AWS identity architecture finalized
- [x] local workforce groups defined
- [ ] AWS permission matrix finalized
- [ ] MFA/session strategy finalized for selected identity source
- [ ] exact AWS integration cost ceiling defined
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

## C — AWS identity foundation
- [ ] verify AWS Organizations state
- [ ] verify IAM Identity Center instance/state
- [ ] confirm supported identity-source integration path from current AWS docs
- [ ] confirm `us-east-1` pricing before any paid directory resource
- [ ] document final integration ADR
- [ ] configure identity source/integration

## D — Workforce authorization
- [ ] create Cloud Admin permission set
- [ ] create DevOps permission set
- [ ] create Developer permission set
- [ ] create Security permission set
- [ ] create Auditor permission set
- [ ] configure account assignments
- [ ] verify group-to-permission mappings

## E — Authentication
- [ ] validate AWS access portal
- [ ] validate SSO login
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
- [ ] final AWS architecture diagram
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

## K — Cleanup / continuity
- [ ] delete paid temporary directory/integration resources if not required later
- [x] retain local identity VMs powered off when not required
- [ ] preserve required workforce identity configuration for Phase 05+ only if cost-safe
- [ ] final AWS cost/resource audit
- [ ] update master transformation repository
- [ ] Phase 04 closeout decision recorded
