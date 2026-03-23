---
title: Tenant Isolation
nav_order: 4
has_children: true
---

# Tenant Isolation

Tenant isolation is enforced at the infrastructure level, not by application
logic. Three mechanisms work together:

- **Impersonation** — the provisioner applies manifests by impersonating the
  tenant's deployer SA. The Kubernetes API server enforces the SA's permissions.
- **Gatekeeper namespace prefix enforcement** — each deployer SA can only create
  or modify resources in namespaces matching its tenant prefix. This is a
  cluster-wide policy, not per-tenant configuration.
- **RBAC escalation prevention** — no SA can be granted broader permissions than
  the deployer SA that created it.

These controls ensure that a tenant cannot access another tenant's namespaces,
resources, or RBAC regardless of what manifests they provision. The event-driven
pipeline's storage and event bus isolation is covered separately under
[Event-Driven Security](event-driven/security.md).
