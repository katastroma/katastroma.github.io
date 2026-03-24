---
title: Multi-Tenancy
nav_order: 2
has_children: true
---

# Multi-Tenancy

Each tenant gets a Kubernetes namespace, a deployer ServiceAccount, and
cluster-scoped RBAC (ClusterRole + ClusterRoleBinding). All tenant resources are
labeled with the tenant identity (`katastroma.org/tenant`) — this label is the
ownership record used for provisioning, pruning, querying, and isolation
enforcement.

Gatekeeper enforces label-based namespace ownership so that each tenant's
deployer SA can only operate in namespaces labeled with its tenant identity. See
[Tenant Isolation](multi-tenancy/tenant-isolation.md) for the full trust chain.
