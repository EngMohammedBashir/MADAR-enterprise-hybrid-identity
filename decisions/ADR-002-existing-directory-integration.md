# ADR-002 — Reuse the Existing Corporate Directory

**Status:** Accepted and implemented

## 🎯 Context

The MADAR environment already contains workforce identities organized in the `madar.local` Active Directory domain.

Introducing a second directory only for AWS would create duplicate employee records and separate identity lifecycle paths for onboarding, role changes and offboarding.

## 🧭 Decision

Represent the corporate identity source with the VMware-hosted Microsoft Active Directory lab and integrate it with AWS through routed WireGuard connectivity and AWS Directory Service AD Connector.

Amazon WorkSpaces Personal consumes the connected directory and provides the end-to-end domain authentication proof.

## 🏗️ Implemented path

```text
MADAR-DC01
madar.local
     ↓
MADAR-WG01
     ↓
Encrypted WireGuard tunnel
     ↓
AWS EC2 WG-HUB
     ↓
AWS Directory Service AD Connector
     ↓
Amazon WorkSpaces
     ↓
madar\sara.ibrahim
```

## 🔎 Why AD Connector

AD Connector provides AWS services with access to the existing Active Directory without turning the lab into a second independently managed directory environment.

For this scenario, that preserves the architectural requirement that `madar.local` remains authoritative while AWS consumes the directory remotely.

## 🧪 Validation

The completed implementation demonstrated:

- AD Connector successfully reaching `madar.local` over the hybrid route,
- an Amazon WorkSpace creating a computer object inside the corporate directory,
- WorkSpace computer `WSAMZN-I0F8R2FL` joined to `MADAR.LOCAL`,
- `madar\sara.ibrahim` successfully authenticating to the WorkSpace,
- DNS connectivity from the WorkSpace to `MADAR-DC01`,
- loss and recovery of that connectivity when the WireGuard dependency was intentionally interrupted and restored.

## 🛡️ Guardrail

The AWS account remained within the agreed Free Plan constraints. Account-plan or AWS Organizations changes were not performed merely to force an IAM Identity Center implementation.

Direct AWS-account SSO therefore remains explicitly documented as a future production extension rather than being presented as a completed Phase 04 capability.

## ⚖️ Consequences

### Benefits

- Corporate identity remains authoritative in one directory.
- AWS consumes real existing users rather than cloud-only duplicates.
- WorkSpaces provides observable end-to-end authentication evidence.
- Identity lifecycle continues to originate from the corporate directory.
- The architecture creates a realistic hybrid dependency that can be failure-tested.

### Trade-offs

- Directory availability from AWS depends on the routed hybrid connection.
- AD Connector depends on healthy AD DNS and required directory ports.
- Temporary AWS integration infrastructure must be cleaned up after the lab.

## ✅ Result

The same corporate employee identity used in the local Active Directory environment was successfully consumed by an AWS-managed desktop through AD Connector, validating the existing-directory integration strategy.