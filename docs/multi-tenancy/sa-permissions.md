---
title: SA Permissions
parent: Multi-Tenancy
nav_order: 4
---

# Tenant SA Permissions

The deployer SA is used by histia for provisioning. Its ClusterRole grants
`create`, `patch`, and `delete`. Cross-tenant read isolation is enforced by the
absence of RoleBindings.

## Tenant Workload SA Permissions

Tenants can provision SAs in their workload namespaces as part of their
manifests — for interactive access (bastion SAs), for workload identity, or any
other purpose. Histia deploys these like any other resource.

Any SA a tenant provisions is isolated by three layers:

1. Kubernetes RBAC — RoleBindings are namespace-scoped. An SA in `acme-prod` has
   zero permissions in any other namespace unless a RoleBinding exists for it
   there.
2. Gatekeeper prefix enforcement — the deployer SA can only create SAs and
   RoleBindings in the tenant's own prefixed namespaces. A tenant cannot
   provision RBAC in another tenant's namespace.
3. RBAC escalation prevention — no SA can be granted broader permissions than
   the deployer SA that created it.
