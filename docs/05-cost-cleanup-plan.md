# Phase 04 Cost & Cleanup Closeout

## 🏁 Final status

**Status:** ✅ COMPLETE  
The temporary AWS infrastructure used for Phase 04 was removed after validation and evidence capture.

## 💰 Cost objective

The lab was designed to demonstrate a realistic hybrid enterprise identity pattern while controlling temporary AWS spend.

The AWS account remained within the agreed account/cost guardrails throughout execution. Paid or potentially billable resources were retained only long enough to complete the required engineering tests and capture evidence.

## 🧾 Final Cost Explorer checkpoint

At final closeout:

```text
Gross month-to-date Usage/Fee : $ 2.1355 USD
AWS credits applied           : $-2.1355 USD
Calculated net                : $ 0.0000 USD
```

These values are **account-level month-to-date Cost Explorer figures**. They must not be interpreted as a precise Phase 04-only cost allocation because other MADAR lab activity occurred in the same account during the month.

The cleanup audit below is the authoritative proof that the temporary Phase 04 resources were removed.

## 🧹 Executed cleanup sequence

The actual destructive closeout followed dependency order:

```text
Amazon WorkSpace
      ↓
WorkSpaces directory registration
      ↓
AD Connector
      ↓
WG-HUB EC2 / Elastic IP
      ↓
Hybrid routes / Security Group
      ↓
Subnets / custom route tables
      ↓
Internet Gateway
      ↓
Phase 04 VPC
```

### Amazon WorkSpaces

Historical resource:

```text
WorkSpace ID : ws-49q8s94dl
User         : sara.ibrahim
Private IP   : 10.50.13.89
```

Termination request:

```bash
AWS_PAGER="" aws workspaces terminate-workspaces \
  --region us-east-1 \
  --terminate-workspace-requests '[{"WorkspaceId":"ws-49q8s94dl"}]' \
  --output json
```

Successful API response:

```json
{
  "FailedRequests": []
}
```

The WorkSpace progressed through `TERMINATING` and subsequently disappeared from `describe-workspaces` output.

### WorkSpaces registration / AD Connector

The directory was deregistered only after the WorkSpace disappeared. AD Connector `d-90667da553` was then deleted and later disappeared from Directory Service inventory.

### WG-HUB EC2 and Elastic IP

Historical routing appliance:

```text
Instance ID : i-029deb16c4c36fd11
Private IP  : 10.50.1.132
Public IP   : 34.228.95.241
```

Final EC2 audit showed:

```text
State : terminated
```

The Phase 04 Elastic IP was released. A subsequent lookup of its allocation ID returned `InvalidAllocationID.NotFound`.

### Network cleanup

The Phase 04-specific hybrid route was removed before deleting dependent network resources.

The following were then removed:

- WG-HUB Security Group,
- private/public subnet associations,
- Phase 04 subnets,
- custom private route table,
- custom public route table,
- Internet Gateway attachment,
- Internet Gateway,
- `MADAR-P04-VPC`.

Final VPC verification returned:

```text
InvalidVpcID.NotFound
```

This is the expected successful post-deletion state.

## 🔎 Final AWS inventory audit

Final live checks confirmed:

```text
Amazon WorkSpaces          none
Directory Service          none
WG-HUB EC2                 terminated
Elastic IPs                none
MADAR Phase 04 VPC         none
```

The terminated EC2 record can remain visible temporarily in EC2 history. A `terminated` instance is not a running compute resource.

## 📸 Final closeout evidence

![Phase 04 final closeout](../evidence/phase04-final-closeout-evidence.png)

The screenshot combines the resource audit with the final Cost Explorer checkpoint.

## 🏠 Local lab continuity

The local VMware machines may remain **powered off** for future MADAR phases:

```text
MADAR-DC01
MADAR-WG01
MADAR-CLIENT01
```

They preserve the corporate identity storyline without creating ongoing AWS infrastructure cost.

## 🔐 Cleanup rule

For future rebuilds:

> Capture proof first. Delete from the application/service layer downward. Verify deletion after every destructive step. Never treat an empty CLI response as an error when the command is explicitly checking that a deleted resource no longer exists.

The full rebuild and destructive cleanup command sequence is retained in the project runbook.
