# Runbooks

Step-by-step operational procedures. Runbooks are living documents:
**if a step is wrong during an incident, fixing the runbook is part of
resolving the incident.**

Most are **Draft** until validated against the real repos/platform as
they come online — a runbook leaves Draft the first time it is executed
successfully end-to-end.

| Runbook | Use when | Status |
|---|---|---|
| [RB-001](RB-001-deploy-a-service.md) — Deploy a service | Changing an existing service | Draft |
| [RB-002](RB-002-add-a-new-service.md) — Add a new service | New stack intake | Draft |
| [RB-003](RB-003-restore-single-service.md) — Restore single service | One service's data lost/corrupt | Draft |
| [RB-004](RB-004-full-disaster-recovery.md) — Full disaster recovery | Pi dead/lost — bare metal rebuild | Draft |
| [RB-005](RB-005-rotate-secrets.md) — Rotate secrets | Scheduled rotation or leak | Draft |
| [RB-006](RB-006-update-services.md) — Update services | Monthly maintenance window | Draft |
| [RB-007](RB-007-add-vpn-peer.md) — Add VPN peer | Enrolling a new device | Draft |
| [RB-008](RB-008-trust-selfsigned-cert.md) — Trust the TLS cert | New device, per OS | Draft |

Authoring rules: [STD-004](../standards/STD-004-documentation-standards.md)
(imperative steps, copy-pasteable commands, explicit preconditions).
