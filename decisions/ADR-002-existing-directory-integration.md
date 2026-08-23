# ADR-002 — Reuse the Existing Corporate Identity Source

**Status:** Planned; exact AWS integration path pending execution-time verification

## Context

The MADAR scenario already contains employee identities before Phase 04. Creating a second unrelated user directory purely for AWS would weaken the enterprise story and create duplicate lifecycle management.

## Decision

Represent the corporate identity source with a small VMware-hosted Microsoft Active Directory lab and integrate that source with the AWS workforce-access design using a currently supported AWS path.

## Guardrail

No paid AWS Directory Service component will be created until the supported integration architecture and current `us-east-1` pricing are verified.

## Consequences

- the project demonstrates federation/directory integration rather than duplicate users,
- the same employee identities can continue into later MADAR phases,
- the local VM can be retained powered off when not in use,
- paid integration infrastructure can be removed when the lab is complete if continuity does not require it.
