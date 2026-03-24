---
title: Namespace Hierarchy
parent: Multi-Tenancy
nav_order: 3
---

# Namespace Hierarchy

Tenants can create child tenants as part of onboarding, authenticated and
authorized through grammateus's API. Child namespaces have an ownerReference
pointing to their parent namespace. Grammateus uses ownerReferences to track the
hierarchy for queries, offboarding, and authorization.

Kubernetes namespaces have no knowledge of each other. There is no parent-child
visibility at the Kubernetes level — each tenant's deployer SA is isolated by
the same [label-based ownership model](tenant-isolation.md) regardless of
hierarchy. Parent-to-child visibility is an [authorization](authentication.md)
concern handled by the platform API.
