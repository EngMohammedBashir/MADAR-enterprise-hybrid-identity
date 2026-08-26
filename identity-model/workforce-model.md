# MADAR Workforce Identity Model

## Implemented local directory model

The local Active Directory lab contains five synthetic employees, one per representative department.

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
    ├── WorkSpaces
    └── Groups
        ├── GG-Management
        ├── GG-IT
        ├── GG-Finance
        ├── GG-HR
        └── GG-Sales
```

The departmental OU answers **where an identity is organized**. The security group answers **which authorization set the user belongs to**. The dedicated `WorkSpaces` OU is for cloud-desktop computer objects, not for moving the employee identity itself.

## Local authorization proof

The IT identity proved group-based authorization before AWS integration:

```text
Sara Ibrahim
   ↓
GG-IT
   ├── IT share       → allowed
   └── Finance share  → denied
```

## AWS consumption proof

Sara's identity remained in the existing IT OU:

```text
CN=Sara Ibrahim,OU=IT,OU=MADAR,DC=madar,DC=local
```

The AWS-managed WorkSpace computer joined separately into:

```text
CN=WSAMZN-I0F8R2FL,OU=WorkSpaces,OU=MADAR,DC=madar,DC=local
```

This separation mirrors a real directory:

```text
User object     = Sara's corporate identity
Computer object = Sara's managed workstation
```

The validated cloud flow is:

```text
sara.ibrahim
   ↓
Amazon WorkSpaces
   ↓
AD Connector
   ↓
madar.local
   ↓
Successful domain authentication
```

## Service account

`svc-adconnector` is a dedicated integration identity used by AWS Directory Service / WorkSpaces for directory operations such as joining WorkSpaces computers into the delegated OU.

It is not an employee account and is not used for interactive workforce login.

## Future authorization extension

The original plan included IAM Identity Center permission sets. That branch was not implemented under the Free Plan account guardrail.

For a future production organization, the same source identities/groups can be extended toward centralized AWS-account SSO and temporary role sessions. That future model is documented separately and is not claimed as completed in this lab.
