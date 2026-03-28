---
title: Onboarding
parent: Multi-Tenancy
nav_order: 1
---

# Onboarding

1. Tenant registers through the API (or via
   [prora](https://github.com/katastroma/prora))
2. [Grammateus](https://github.com/katastroma/grammateus) creates a namespace
   for the tenant and labels it with the tenant identity (see
   [Trust Chain](tenant-isolation.md#trust-chain) for why this ordering matters)
3. Grammateus creates a deployer ServiceAccount in the tenant namespace (see
   [SA Permissions](sa-permissions.md) for the permission model)
4. Tenant configures the platform source handlers, source targets, any necessary
   credentials with the source handler APIs
5. Tenant configures their sources with any verifications needed to wire up
   sending events from their sources to the source handler APIs

All tenant resources — namespace, ServiceAccount, ClusterRole, and
ClusterRoleBinding from grammateus, source credentials and webhook secrets from
source handler APIs, etc. — are labeled with the tenant identity. This allows
histia to find and prune all tenant resources (including cluster-scoped ones)
during offboarding.
