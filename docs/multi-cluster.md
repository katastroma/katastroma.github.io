---
title: Multi-Cluster
nav_order: 3
---

# Multi-Cluster

**Not yet designed.**

Tenants provide the target cluster during onboarding, along with access
credentials. The platform provisions resources to that cluster by passing a
cluster identity (server address and credentials) to the provisioner.

Open questions:

- Pipeline services (renderer, orderer, provisioner) may need to run on the
  target cluster. Whether this is a prerequisite or handled during onboarding
  depends on the access model.
- Cluster identity may need to be tracked in the tenant namespace.
