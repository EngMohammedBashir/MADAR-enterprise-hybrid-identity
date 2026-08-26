# ADR-001 — Centralized Workforce Identity

**Status:** Accepted and implemented for hybrid directory authentication  
**Direct AWS-account SSO:** Future production extension

## 🎯 Context

MADAR already has a corporate identity authority: Microsoft Active Directory domain `madar.local`.

Creating unrelated AWS-side employee identities would introduce duplicate accounts, fragmented lifecycle management and a weaker enterprise identity model.

The architecture therefore needed to preserve the existing directory as the authoritative identity source while proving that an AWS-managed end-user service could consume those identities across the hybrid boundary.

## 🧭 Decision

Retain `madar.local` as the source of workforce identity and integrate AWS with the existing directory rather than duplicating employee accounts.

The implemented Phase 04 validation path is:

```text
Corporate employee
      ↓
madar.local Active Directory
      ↓
WireGuard routed hybrid connectivity
      ↓
AWS Directory Service AD Connector
      ↓
Amazon WorkSpaces
      ↓
Domain authentication
```

## 🧪 Implemented proof

The test identity `sara.ibrahim` remained an on-premises Active Directory account and successfully authenticated to an AWS WorkSpace joined to `MADAR.LOCAL`.

This validates centralized identity consumption across the hybrid path without creating a second employee identity solely for the cloud desktop.

## ☁️ Direct AWS-account SSO

IAM Identity Center remains the preferred future direction for centralized AWS-account workforce access, temporary sessions and permission-set-based authorization.

That branch was not represented as implemented in this lab. The AWS account remained within the agreed Free Plan guardrails and Organizations/account-plan changes were not forced solely to satisfy the original design.

## ⚖️ Consequences

### Benefits

- One authoritative employee identity source.
- Reduced duplicate-account lifecycle management.
- Real hybrid authentication proof through an AWS-managed service.
- Clear separation between authentication source and cloud service consumption.
- No false claim that direct AWS-account SSO was implemented when it was not.

### Trade-offs

- Availability of AWS-side directory authentication depends on the hybrid network path and on-premises AD/DNS health.
- Direct AWS-account workforce SSO remains an additional production architecture step.

## ✅ Result

Phase 04 successfully demonstrated that the existing `madar.local` workforce identity can be consumed from AWS through AD Connector and Amazon WorkSpaces while keeping the corporate directory authoritative.