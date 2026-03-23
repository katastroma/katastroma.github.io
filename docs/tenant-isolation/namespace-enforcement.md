---
title: Namespace Enforcement
parent: Tenant Isolation
nav_order: 2
---

# Namespace Enforcement

Gatekeeper enforces namespace prefix conventions. Each tenant's deployer SA can
only create namespaces matching the tenant's prefix (e.g. `acme-*`). This
prevents namespace collisions between tenants.

The Gatekeeper policy derives the tenant prefix from the SA identity. The SA
username in Kubernetes is `system:serviceaccount:<namespace>:<name>`. The policy
validates:

- The SA namespace starts with `tenant-` (anchors the SA to a tenant namespace)
- The SA name prefix matches the namespace suffix (`tenant-acme` →
  `acme-deployer` → prefix `acme`)

This prevents rogue SAs outside `tenant-*` namespaces from getting prefix
enforcement. Grammateus must follow the naming convention `<prefix>-deployer`
when creating SAs.

**Open issue:** This relies on no entity other than grammateus being able to
create SAs in `tenant-*` namespaces. Without this restriction, a tenant's
deployer SA (or a controller it deploys) could create a rogue SA in
`tenant-acme` with a different name — Gatekeeper would derive a different prefix
from that SA, allowing it to operate outside the tenant's intended namespace
scope. A Gatekeeper policy restricting SA creation in `tenant-*` namespaces to
grammateus's own SA would close this loop.
