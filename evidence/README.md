# Phase 04 Evidence Index

This folder contains evidence that proves an identity, policy or authorization claim. Screenshots are kept with descriptive filenames and are linked from the project README where useful.

## Local identity / Active Directory evidence — captured

| Evidence file | What it proves |
|---|---|
| `legacy-vm-running-before-identity-modernization.png` | representative legacy environment remained available before the identity lab work |
| `Domain-Controller-Verification.png` | `MADAR-DC01`, `madar.local` and domain-controller role verification |
| `Active-Directory-Domain-Controller.png` | `MADAR-DC01` visible inside Active Directory as the domain controller |
| `Active-Directory-OU-Structure.png` | MADAR departmental OU structure |
| `Active-Directory-Security-Groups.png` | Global Security Groups for the workforce departments |
| `Manual-AD-User-Creation.png` | first synthetic user created manually to learn the GUI workflow |
| `PowerShell-Automated-AD-Users-Creation.png` | repeat user creation validated through PowerShell automation |
| `AD-Security-Group-Membership-Verification.png` | each synthetic user mapped to the intended departmental security group |
| `GPO-IT-Domain-Firewall-Policy.png` | configured IT Domain firewall policy |
| `CLIENT01-System-Verification.png` | Windows client baseline / hostname verification |
| `CLIENT01-Domain-Join-Success.png` | `MADAR-CLIENT01` successfully joined `madar.local` |
| `Domain-User-Login-Sara.png` | domain-user authentication on the client |
| `GPO-IT-Security-Applied.png` | `GPO-IT-Security` applied to the domain client |
| `Domain-Firewall-GPO-Verification.png` | Domain firewall enforcement verified on the client |
| `Sara-IT-Share-Access-Success.png` | intended IT-share access succeeded for the IT identity |
| `Sara-Finance-Access-Denied.png` | cross-department Finance-share access was denied as designed |

## Local evidence story

```text
Domain Controller
      ↓
OU + Users + Groups
      ↓
Domain Client Join
      ↓
Domain User Login
      ↓
GPO Enforcement
      ↓
Allowed IT Access ✅
Denied Finance Access ❌
```

The denied-access screenshot is a **success condition**: it demonstrates that group membership produces a real authorization boundary.

## AWS evidence — pending

The next evidence set will be created during the AWS integration gate and should cover:

```text
AWS identity architecture / integration
IAM Identity Center state
Permission sets
Account assignments
SSO login
MFA proof
Temporary console / CLI session
Allowed AWS action
Denied AWS action
CloudTrail audit evidence
Joiner / mover / leaver revocation
Final cost and cleanup audit
```

## Evidence rule

Screenshots must not expose passwords, MFA seeds, private keys, reusable access keys, session tokens or other secrets. Lab passwords are intentionally excluded from repository documentation.
