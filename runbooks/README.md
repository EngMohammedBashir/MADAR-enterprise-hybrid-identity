# 📚 MADAR Phase 04 — Runbook Library

This folder is the operational handbook for rebuilding and validating the Phase 04 hybrid-identity environment.

## 🧭 Start here

| Runbook | Use it when |
|---|---|
| [`00-lab-rebuild-and-validation.md`](00-lab-rebuild-and-validation.md) | You want to rebuild the lab from scratch, understand every validation gate, troubleshoot failures, run the VPN failure/recovery drill, or perform cleanup. This is the **full detailed handbook**. |
| [`01-command-check-reference.md`](01-command-check-reference.md) | You already understand the architecture and only need the important commands/checks quickly. |
| [`onboard-user.md`](onboard-user.md) | You are documenting a joiner/onboarding identity workflow. |
| [`change-user-role.md`](change-user-role.md) | You are documenting a mover/group-role change workflow. |
| [`offboard-user.md`](offboard-user.md) | You are documenting a leaver/access-revocation workflow. |

## 🧠 Recommended learning order

```text
1. README.md at repository root
       ↓
2. docs/03-target-architecture.md
       ↓
3. 00-lab-rebuild-and-validation.md
       ↓
4. docs/13-hybrid-connectivity-troubleshooting.md
       ↓
5. evidence/README.md
       ↓
6. 01-command-check-reference.md for future repetitions
```

## 🎯 Philosophy

The runbooks are written so that the lab is not a collection of copied commands.

Every important check should answer:

```text
What am I checking?
Why does it matter?
What does success look like?
What does failure mean?
Which layer should I inspect next?
```

The central troubleshooting rule is:

```text
Observe → isolate one layer → test → change one thing → re-test
```

Avoid changing routing, Security Groups, WireGuard, Windows Firewall and credentials simultaneously. If several layers change at once, a later success is difficult to explain or reproduce.

## 🔐 Secret-handling rule

Never commit:

- AD passwords,
- AWS access keys,
- AWS session tokens,
- WireGuard private keys,
- MFA seeds,
- reusable cookies or registration secrets.

Public architecture identifiers and synthetic lab names may be documented when useful for reproducibility.
