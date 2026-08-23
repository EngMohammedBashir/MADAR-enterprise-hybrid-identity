# Phase 04 Cost & Cleanup Plan

## Cost objective

Keep the identity project inexpensive while still demonstrating a realistic enterprise pattern.

## Expected low/no direct service-cost components

Core IAM, STS, AWS Organizations and IAM Identity Center workforce-access functions do not require a separate standalone service fee. Normal charges can still arise from services accessed during testing.

## Cost-sensitive component

Directory integration may require an AWS Directory Service resource with hourly charges. Therefore:

```text
Before create
  -> verify current AWS docs
  -> verify supported architecture
  -> verify us-east-1 price
  -> calculate test-window cost
  -> set deletion checkpoint
```

No paid directory resource should be left running merely to preserve the portfolio story.

## Continuity after Phase 04

The local `MADAR-DC01` VMware VM may be retained powered off for later phases so the same workforce identities can be reused.

AWS-side configuration should be retained only when it is both useful for Phase 05+ continuity and cost-safe. Paid temporary integration infrastructure should be removed after evidence unless explicitly justified.

## Final audit

Before closing Phase 04, review:

- Directory Service resources,
- EC2/ENI/network dependencies introduced only for identity integration,
- CloudWatch/logging resources created by the lab,
- temporary policies/roles,
- unexpected IAM users or long-lived access keys,
- any other paid resource created during troubleshooting.
