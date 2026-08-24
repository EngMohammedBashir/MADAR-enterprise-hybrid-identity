# MADAR Workforce Identity Model

## Implemented local directory model

The local Active Directory lab now contains five synthetic employees, one per representative department.

| Department | User | Logon name | AD security group |
|---|---|---|---|
| Management | Ahmed Ali | `ahmed.ali` | `GG-Management` |
| IT | Sara Ibrahim | `sara.ibrahim` | `GG-IT` |
| Finance | Omar Hassan | `omar.hassan` | `GG-Finance` |
| HR | Noura Saleh | `noura.saleh` | `GG-HR` |
| Sales | Khalid Mansour | `khalid.mansour` | `GG-Sales` |

All users are synthetic. No real employee identities are used.

## Directory organization

```text
madar.local
└── MADAR
    ├── Management
    ├── IT
    ├── Finance
    ├── HR
    ├── Sales
    ├── Users
    ├── Computers
    └── Groups
        ├── GG-Management
        ├── GG-IT
        ├── GG-Finance
        ├── GG-HR
        └── GG-Sales
```

The departmental OU answers **where the identity or computer is organized**. The security group answers **which authorization set the identity belongs to**. These are deliberately separate concepts.

## Local authorization proof

The IT identity was used to prove group-based authorization before AWS integration:

```text
Sara Ibrahim
   ↓
GG-IT
   ├── IT share       → allowed
   └── Finance share  → denied
```

This is the source-side proof that identity and group membership have operational meaning rather than existing only as directory objects.

## AWS authorization model — next gate

The local departmental groups are the **source identity model**. AWS permission sets will be designed separately around AWS job functions and least privilege. A one-to-one mapping from every AD department to an AWS administrator role is not assumed.

Target flow:

```text
Employee
   ↓
Corporate identity / group membership
   ↓
Supported AWS identity integration
   ↓
IAM Identity Center
   ↓
AWS permission set
   ↓
AWS account assignment
   ↓
Temporary SSO session
```

Planned AWS access personas remain Cloud Admin, DevOps, Developer, Security and Auditor; the exact mapping from the implemented source identities/groups will be finalized during the AWS integration gate.

## Lifecycle model

```text
Joiner  → create/enable identity → group → AWS access
Mover   → change group/assignment → permission change
Leaver  → disable/remove → AWS access revoked
```
