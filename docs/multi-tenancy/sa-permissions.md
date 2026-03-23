---
title: SA Permissions
parent: Multi-Tenancy
nav_order: 4
---

# SA Permissions

## Deployer SA

Grammateus creates a deployer SA per tenant during onboarding with:

- A ClusterRole granting `create`, `patch`, and `delete` on all resources (`*`)
  — `create` and `patch` for SSA, `delete` for pruning
- A ClusterRoleBinding binding the SA to the ClusterRole
- Gatekeeper constrains where the SA can operate by
  [namespace prefix](../tenant-isolation/namespace-enforcement.md)

No `read` permissions — this prevents a tenant's deployer SA from querying
resources in other tenants' namespaces. Combined with Gatekeeper prefix
enforcement (tenants can only provision resources in their own prefixed
namespaces — there are no shared namespaces), this enforces complete
cross-tenant read isolation at the RBAC level.

Histia uses the deployer SA via
[impersonation](../tenant-isolation/impersonation.md) when provisioning.

## Workload SA Permissions

Tenants can provision SAs in their workload namespaces as part of their
manifests — for interactive access (bastion SAs), for workload identity, or any
other purpose. Histia deploys these like any other resource.

Any SA a tenant provisions is isolated by three layers:

1. Kubernetes RBAC — RoleBindings are namespace-scoped. An SA in `acme-prod` has
   zero permissions in any other namespace unless a RoleBinding exists for it
   there.
2. [Gatekeeper prefix enforcement](../tenant-isolation/namespace-enforcement.md)
   — the deployer SA can only create SAs and RoleBindings in the tenant's own
   prefixed namespaces.
3. RBAC escalation prevention — no SA can be granted broader permissions than
   the deployer SA that created it.
