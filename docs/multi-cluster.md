---
title: Multi-Cluster
nav_order: 5
---

# Multi-Cluster

**Open issue:** Should be a straightforward solve but may have to track
ClusterIdentity inside the tenant namespace.

The architecture should support:

- during tenant onboarding, tenant will provide what cluster the platform should
  install resources to and provide access to that cluster.

This will likely require bringing up the source handler services (including the
renderer, orderer, and provisioner implementations) to those clusters
beforehand(?). Otherwise tenant onboarding could possibly do it given sufficient
access/permissions.

Once those prerequisites are installed, provisioning to that cluster for a watch
target is just a matter of passing a ClusterIdentity (cluster server and
credentials) to the provisioner for it to provision.
