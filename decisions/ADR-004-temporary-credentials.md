# ADR-004 — Temporary Workforce Credentials

**Status:** Accepted for Phase 04 design

## Decision

Ordinary MADAR workforce access to AWS will use temporary SSO sessions rather than employee-specific long-lived AWS access keys.

## Why

Temporary sessions reduce credential persistence, align naturally with centralized SSO, and make role changes/offboarding easier to enforce.

## Validation

Phase 04 will prove both console SSO and AWS CLI SSO, inspect the resulting session identity, and document that the employee workflow does not depend on static access keys.
