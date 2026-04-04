---
title: Provisioner
parent: Event-Driven
nav_order: 5
---

# Provisioner

The provisioner receives manifests via gRPC streaming and applies them to the
cluster using server-side apply. It
[impersonates](../multi-tenancy/tenant-isolation.md#impersonation) the tenant's
provisioner ServiceAccount so that Kubernetes evaluates authorization against
the tenant's permissions, not the platform's.

Before applying, the provisioner verifies each manifest's resource type supports
the `list` verb via the Kubernetes discovery API. Resource types that cannot be
listed are rejected — the [pruner](pruner.md) relies on listing by label to
calculate prune targets, so any resource that cannot be listed cannot be pruned
and would be orphaned.

The provisioner verifies the
[source target lease](../event-driven.md#source-target-leasing) before
applying.

The provisioner streams applied manifests to the [pruner](pruner.md).

The [katartismos](https://github.com/katastroma/katartismos) interface defines
the gRPC contract.
