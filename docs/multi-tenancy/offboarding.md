---
title: Offboarding
parent: Multi-Tenancy
nav_order: 2
---

# Offboarding

1. Grammateus walks the tenant hierarchy bottom-up, telling histia to prune all
   resources labeled with each child tenant's identity (deepest children first)
2. Grammateus tells histia to prune all resources labeled with the root tenant's
   identity

All tenant resources — namespaces, workload namespaces, ClusterRoles,
ClusterRoleBindings, Secrets, workloads — are deleted by the prune because they
are all labeled with the tenant identity.
