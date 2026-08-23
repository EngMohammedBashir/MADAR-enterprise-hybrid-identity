# ADR-003 — Group-Based Authorization

**Status:** Accepted for Phase 04 design

## Decision

Assign AWS access through workforce groups mapped to permission sets instead of managing normal access directly per user.

## Why

A group model mirrors how real organizations manage teams and makes joiner/mover/leaver operations easier to test and audit.

## Expected groups

- Cloud-Admins
- DevOps-Engineers
- Developers
- Security-Team
- Auditors

## Validation

Each group must demonstrate intended allowed behavior and at least one denied action that proves the privilege boundary.
