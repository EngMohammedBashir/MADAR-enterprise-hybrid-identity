# Phase 04 Cost & Cleanup Plan

## Cost objective

Keep the identity project inexpensive while still demonstrating a realistic hybrid enterprise pattern.

## Account guardrail

The AWS account remained on the **AWS Free Plan** throughout the lab.

The project explicitly rejected any step that would require upgrading the account merely to continue an architectural branch.

At the WorkSpaces validation checkpoint:

```text
Credits issued              : $180.00
Estimated used              : $3.53
Estimated remaining         : $176.47
Amazon WorkSpaces           : listed as an applicable credit product
Selected WorkSpaces bundle  : Free tier eligible
```

## Cost-sensitive components used

### AWS Directory Service AD Connector

The Connector was used only for the hybrid directory integration and must not be left running indefinitely after acceptance unless a later phase explicitly needs it.

### EC2 WireGuard routing appliance

The EC2 `MADAR-P04-WG-HUB` is a temporary network appliance used to terminate the routed WireGuard tunnel. Stop it whenever the hybrid path is not under test.

### Amazon WorkSpaces

Only one WorkSpace was created:

```text
WorkSpace ID : ws-49q8s94dl
Bundle       : Standard Windows — Free tier eligible
Running mode : AutoStop after 1 hour
User         : sara.ibrahim
```

The purpose was end-to-end identity validation, not permanent VDI hosting.

No NAT Gateway was created for the WorkSpaces test.

## Pause procedure

When pausing the lab:

```text
Amazon WorkSpace        -> Stop
AWS WG-HUB EC2          -> Stop
MADAR-DC01 VMware VM    -> Shutdown
MADAR-WG01 VMware VM    -> Shutdown
MADAR-CLIENT01          -> Keep powered off unless needed
AD Connector            -> no Stop state; retain only until final cleanup decision
```

## Final cleanup sequence

After documentation/evidence review:

1. verify all required screenshots are committed,
2. stop and then delete the test WorkSpace if no further proof is needed,
3. deregister the WorkSpaces directory when safe,
4. delete AD Connector when no later phase depends on it,
5. terminate the temporary WireGuard EC2 routing appliance,
6. release any public IPv4 / Elastic IP that is no longer required,
7. remove Phase 04-specific route-table entries and temporary security rules if the VPC will not be reused,
8. delete temporary subnets only after checking dependencies,
9. power off local VMware VMs,
10. review Bills / Credits / Cost Explorer,
11. verify no unintended Phase 04 paid resource remains.

## Continuity after Phase 04

`MADAR-DC01` and the local identity VMs may be retained **powered off** so later phases can reuse the same corporate identity story without rebuilding the directory.

AWS-side resources should remain only when their continuity value is greater than their ongoing cost.

## Final audit checklist

Review:

- WorkSpaces Personal resources,
- registered WorkSpaces directories,
- Directory Service resources,
- EC2 / ENIs / public IPv4 introduced for the hybrid router,
- route tables and Security Groups created for Phase 04,
- CloudWatch/logging resources created by the lab,
- unexpected IAM users or long-lived access keys,
- Bills / Credits after cleanup.
