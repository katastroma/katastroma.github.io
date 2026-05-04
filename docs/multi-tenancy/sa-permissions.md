---
title: SA Permissions
parent: Multi-Tenancy
nav_order: 4
---

# SA Permissions

Two distinct sets of ServiceAccounts exist: **platform SAs** that platform
services run as, and **tenant SAs** that platform services impersonate when
operating on tenant resources.

## Platform SAs

Platform services run with their own SAs in the platform namespace.

**Platform Provisioner SA** has `impersonate` on ServiceAccounts — to
impersonate tenant provisioner SAs for server-side apply operations.

**Platform Pruner SA** has `list` on all resources — to query the cluster by
label for pruning diffs — and `impersonate` on ServiceAccounts — to impersonate
tenant pruner SAs for delete operations.

The labeler has no platform SA — it transforms manifests in-flight and does not
interact with the Kubernetes API.

## Tenant SAs

Grammateus creates two SAs per tenant during onboarding, both in the tenant
namespace, with fixed names.

### Tenant Provisioner SA

Named `provisioner`. Identity:
`system:serviceaccount:<tenant-namespace>:provisioner`.

A ClusterRole granting `create` and `patch` on all resources (`*`) — the verbs
required for server-side apply
([Kubernetes: Server-Side Apply](https://kubernetes.io/docs/reference/using-api/server-side-apply/)).

The platform provisioner impersonates this SA via
[impersonation](tenant-isolation.md#impersonation) when provisioning.

### Tenant Pruner SA

Named `pruner`. Identity: `system:serviceaccount:<tenant-namespace>:pruner`.

A ClusterRole granting `delete` on all resources (`*`).

The platform pruner impersonates this SA via
[impersonation](tenant-isolation.md#impersonation) when pruning.

### Constraints

Neither tenant SA has read permissions — no `get`, `list`, or `watch`. A tenant
SA cannot discover or read resources in any namespace. Cross-tenant read
isolation is enforced by the absence of read verbs, not by namespace boundaries.

Gatekeeper constrains where each SA can operate using label-based namespace
ownership — see [Tenant Isolation](tenant-isolation.md) for the full trust
chain.

## Workload SA Permissions

Tenants can provision SAs in their workload namespaces as part of their
manifests — for interactive access (bastion SAs), for workload identity, or any
other purpose. The provisioner deploys these like any other resource.

Any SA a tenant provisions is isolated by:

1. **Namespace-scoped RBAC** — RoleBindings are namespace-scoped. A workload SA
   has zero permissions in any namespace it doesn't have a RoleBinding for.
2. **Tenant SA containment** — tenant SAs can only create SAs and RoleBindings
   in namespaces labeled with its tenant identity, so workload SAs can only
   exist within the tenant's own namespaces (see
   [Tenant Isolation](tenant-isolation.md)).
3. **RBAC escalation prevention** — no SA can be granted broader permissions
   than the SA that created it. Workload SAs cannot exceed the tenant SA's
   permissions.
