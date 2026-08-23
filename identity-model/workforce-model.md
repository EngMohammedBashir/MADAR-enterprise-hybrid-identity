# MADAR Workforce Identity Model

## Planned teams

| Group | Example employee | Purpose |
|---|---|---|
| Cloud-Admins | admin.user | controlled platform administration |
| DevOps-Engineers | ahmed.devops | infrastructure and delivery operations |
| Developers | sara.dev | application development |
| Security-Team | noura.security | security visibility and investigation |
| Auditors | omar.audit | read-only review and evidence access |

All users are synthetic. No real employee identities are used.

## Authorization principle

```text
Employee
   ↓
Corporate group membership
   ↓
AWS permission set
   ↓
AWS account assignment
   ↓
Temporary SSO session
```

Access is assigned to groups whenever practical rather than directly to individuals.

## Lifecycle model

```text
Joiner  -> create/enable identity -> group -> AWS access
Mover   -> change group -> permission change
Leaver  -> disable/remove -> AWS access revoked
```
