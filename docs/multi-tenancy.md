---
title: Multi-Tenancy
nav_order: 3
has_children: true
---

# Multi-Tenancy

Each tenant gets a Kubernetes namespace, a deployer ServiceAccount, and
cluster-scoped RBAC (ClusterRole + ClusterRoleBinding). All tenant resources are
labeled with the tenant identity — this label is the ownership record used for
provisioning, pruning, querying, and isolation enforcement.

Gatekeeper enforces namespace prefix conventions so that each tenant's deployer
SA can only operate within its own prefixed namespaces (see
[Namespace Enforcement](tenant-isolation/namespace-enforcement.md)). This is the
foundation of tenant isolation — it is not per-tenant configuration but a
cluster-wide policy that applies automatically to every tenant.
