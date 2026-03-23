---
title: Resource Ownership
parent: Tenant Isolation
nav_order: 5
---

# Resource Ownership

Every tenant resource is labeled with the tenant identity. Resources provisioned
by histia are additionally labeled with the platform labels
([see `katartismos`](https://github.com/katastroma/katartismos)). These labels
are the ownership record — used for pruning, querying, and isolation
enforcement.
