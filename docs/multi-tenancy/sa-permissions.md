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

No `read` permissions — deployer SAs have `create`, `patch`, and `delete` only.
No `get`, `list`, or `watch`. A deployer SA cannot discover or read resources in
any namespace. Cross-tenant read isolation is enforced by the absence of read
verbs, not by namespace boundaries.

Gatekeeper constrains where the SA can operate using label-based namespace
ownership — see [Tenant Isolation](tenant-isolation.md) for the full trust
chain.

Histia uses the deployer SA via
[impersonation](tenant-isolation.md#impersonation) when provisioning.

## Workload SA Permissions

Tenants can provision SAs in their workload namespaces as part of their
manifests — for interactive access (bastion SAs), for workload identity, or any
other purpose. Histia deploys these like any other resource.

Any SA a tenant provisions is isolated by:

1. **Namespace-scoped RBAC** — RoleBindings are namespace-scoped. A workload SA
   has zero permissions in any namespace it doesn't have a RoleBinding for.
2. **Deployer SA containment** — the deployer SA can only create SAs and
   RoleBindings in namespaces labeled with its tenant identity, so workload SAs
   can only exist within the tenant's own namespaces (see
   [Tenant Isolation](tenant-isolation.md)).
3. **RBAC escalation prevention** — no SA can be granted broader permissions
   than the deployer SA that created it. Workload SAs cannot exceed the deployer
   SA's permissions.
