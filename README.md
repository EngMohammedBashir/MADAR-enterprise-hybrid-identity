# MADAR — Enterprise Identity & Workforce Access

## Phase 04 of the MADAR Cloud Transformation

> **Status: PLANNED / READY FOR EXECUTION**  
> Corporate Active Directory representation → AWS identity integration → IAM Identity Center → SSO/MFA → permission sets → least-privilege validation → lifecycle testing → audit → cleanup.

MADAR has completed the migration of its representative legacy workload to AWS. Phase 04 addresses the next business problem: **how employees securely access the growing AWS environment using centralized workforce identity instead of individual long-lived IAM credentials.**

## Business story

The MADAR scenario assumes a corporate employee directory already exists outside AWS. For the hands-on lab, a small Windows Server VM will represent that existing corporate Active Directory.

```text
MADAR workforce
      |
      v
Corporate Active Directory
      |
      | identity integration
      v
AWS IAM Identity Center
      |
      +--> Cloud Admin
      +--> DevOps Engineer
      +--> Developer
      +--> Security
      +--> Auditor
      |
      v
AWS account access through SSO + temporary sessions
```

The Active Directory VM is a **lab representation of a pre-existing corporate identity system**. It is not a second workload migration and it is not a claim that the directory was created after Phase 03 in the company narrative.

## Phase objective

Build and validate an enterprise-style workforce-access model that demonstrates:

- centralized employee identity,
- group-based authorization,
- AWS IAM Identity Center,
- SSO and MFA,
- temporary AWS credentials,
- permission sets and least privilege,
- positive and negative access tests,
- console and CLI SSO,
- onboarding, role-change and offboarding workflows,
- audit evidence,
- cost-aware cleanup.

## Planned workforce model

| Team | Intended access posture |
|---|---|
| Cloud Admins | tightly controlled administrative access |
| DevOps Engineers | infrastructure/deployment operations without unrestricted identity administration |
| Developers | application-development access only |
| Security Team | security visibility and investigation access |
| Auditors | read-only evidence/review access |

## Acceptance philosophy

Phase 04 is not complete because Identity Center says `Enabled`.

```text
Authentication works                 != sufficient
Permission set exists                != sufficient
User can open AWS console            != sufficient

SUCCESS = centralized identity
        + SSO/MFA
        + role-based permissions
        + temporary credentials
        + allowed-action proof
        + denied-action proof
        + lifecycle revocation proof
        + audit evidence
        + intentional cleanup
```

## Execution style

The implementation intentionally uses a hybrid learning approach:

```text
New concept        -> AWS Console first
Understand resource -> CLI inspection
Repeated operation -> CLI where useful
Validation          -> CLI + console evidence
Security boundary   -> explicit negative tests
```

## Cost guardrail

IAM, STS, AWS Organizations and IAM Identity Center do not add a standalone service charge for the core workforce-access lab. Directory integration can introduce hourly cost depending on the selected AWS Directory Service path, so no paid directory resource will be created until the exact supported architecture and hourly cost are verified for `us-east-1`.

Any paid integration resource used only for validation will be created late, tested quickly, evidenced, and removed after acceptance unless it is explicitly required by later MADAR phases.

## Planned repository structure

```text
.
├── README.md
├── CURRENT-STATE.md
├── REPOSITORY-SCOPE.md
├── checklists/
│   └── phase04-master-checklist.md
├── decisions/
├── docs/
├── evidence/
│   └── README.md
├── identity-model/
├── policies/
├── runbooks/
└── tests/
```

## Relationship to the MADAR journey

```text
Phase 01  Cloud Foundation                     COMPLETE
Phase 02  Serverless Event Processing          COMPLETE
Phase 03  Legacy Migration & Data Center Exit  COMPLETE
Phase 04  Enterprise Identity & Workforce      READY TO EXECUTE
Phase 05  Application Modernization            FUTURE
```

Master transformation record: [`EngMohammedBashir/MADAR-cloud-transformation`](https://github.com/EngMohammedBashir/MADAR-cloud-transformation)
