# ADR-001 — Centralize Workforce Access

**Status:** Accepted for Phase 04 design

## Context

MADAR's AWS footprint now supports migrated and cloud-native workloads. Managing ordinary employee access with separate IAM users and long-lived credentials would not scale safely.

## Decision

Use a centralized workforce identity pattern based on AWS IAM Identity Center for human AWS access.

## Consequences

- workforce access becomes session-based,
- group/role assignments become the main authorization model,
- employee lifecycle changes can be applied centrally,
- IAM users are not the default workforce pattern,
- later phases can reuse the same identity foundation.
