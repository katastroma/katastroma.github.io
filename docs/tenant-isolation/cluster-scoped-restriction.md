---
title: Cluster-Scoped Restriction
parent: Tenant Isolation
nav_order: 3
---

# Cluster-Scoped Resource Restriction

The deployer SA's ClusterRole uses `*` for resources so tenants can provision
CRs from any CRD. RBAC is additive only — there are no deny rules. Gatekeeper
blocks tenant deployer SAs from creating dangerous cluster-scoped resources:

- ClusterRoles
- ClusterRoleBindings

The policy checks: if the requesting SA is a tenant deployer (namespace starts
with `tenant-`) and the resource is a ClusterRole or ClusterRoleBinding —
reject.

**Open issue:** CRDs are cluster-scoped. Blocking tenant CRD creation prevents
tenants from installing operators or charts that include CRDs. Allowing it means
CRDs are visible cluster-wide to all tenants. CRD groups are defined by the CRD
spec itself — tenants cannot prefix third-party CRD groups without forking.

Candidate mitigations:

- Block tenant CRD creation entirely — tenants request CRDs through the
  platform, which vets and installs them. Safest but adds operational overhead.
- Allow CRD creation but use Gatekeeper to restrict which API groups tenants can
  register. Limits blast radius but requires maintaining an allowlist.
- Allow CRD creation with labeling — all tenant-created CRDs are labeled with
  the tenant identity. Visibility is cluster-wide but ownership is tracked for
  cleanup and auditing.
