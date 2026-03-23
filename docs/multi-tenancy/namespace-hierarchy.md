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
