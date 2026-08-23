# Phase 04 Master Checklist

## A — Planning
- [ ] business problem approved
- [ ] current-state identity model documented
- [ ] target identity architecture documented
- [ ] workforce groups defined
- [ ] permission matrix defined
- [ ] MFA/session strategy defined
- [ ] cost ceiling defined
- [ ] cleanup plan defined

## B — Corporate directory lab
- [ ] create `MADAR-DC01` in VMware
- [ ] install Windows Server
- [ ] install/configure AD DS
- [ ] configure DNS
- [ ] create MADAR domain
- [ ] create organizational units
- [ ] create workforce groups
- [ ] create synthetic employee accounts
- [ ] verify group membership
- [ ] capture directory baseline evidence

## C — AWS identity foundation
- [ ] verify AWS Organizations state
- [ ] verify IAM Identity Center instance/state
- [ ] confirm supported identity-source integration path from current AWS docs
- [ ] confirm `us-east-1` pricing before any paid directory resource
- [ ] document integration ADR
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

## F — Positive access tests
- [ ] Cloud Admin allowed action
- [ ] DevOps allowed action
- [ ] Developer allowed action
- [ ] Security allowed read/investigation action
- [ ] Auditor read-only action

## G — Negative access tests
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
- [ ] architecture diagram
- [ ] AD baseline screenshot
- [ ] users/groups screenshot
- [ ] Identity Center screenshot
- [ ] permission-set screenshot
- [ ] account-assignment screenshot
- [ ] SSO login screenshot
- [ ] MFA proof
- [ ] CLI SSO proof
- [ ] negative-test proof
- [ ] offboarding proof
- [ ] audit proof
- [ ] final cost/cleanup proof

## K — Cleanup / continuity
- [ ] delete paid temporary directory/integration resources if not required later
- [ ] preserve local `MADAR-DC01` VM powered off for later MADAR phases
- [ ] preserve required workforce identity configuration for Phase 05+ only if cost-safe
- [ ] final AWS cost/resource audit
- [ ] update master transformation repository
- [ ] Phase 04 closeout decision recorded
