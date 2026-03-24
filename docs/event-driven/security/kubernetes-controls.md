---
title: Kubernetes-Provided Controls
parent: Security
grand_parent: Event-Driven
nav_order: 1
---

# Kubernetes-Provided Controls

- **ServiceAccount identity** — each service in the pipeline runs with its own
  dedicated SA. SA tokens authenticate to both the event bus and storage via the
  Kubernetes TokenReview API.
- **Gatekeeper restricts tenant deployer SAs to their own namespaces** —
  deployer SAs can only operate in namespaces labeled with their tenant
  identity. The platform namespace has no tenant label — deployer SAs cannot
  create resources there. Tenants cannot deploy services that impersonate
  platform SAs. Tenants have no SA that the event bus or storage would
  recognize. See [Tenant Isolation](../../multi-tenancy/tenant-isolation.md) for
  the full trust chain.
- **RBAC restricts platform Secrets** — event bus and storage admin credentials
  are Kubernetes Secrets in the platform namespace, accessible only to platform
  SAs. Compromise of admin credentials is a full platform compromise scenario.
- **Gatekeeper prevents lateral movement** — a compromised service in the
  pipeline cannot create resources in tenant namespaces, cannot escalate its
  RBAC permissions, and cannot create new SAs.
