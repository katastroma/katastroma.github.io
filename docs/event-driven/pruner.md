---
title: Pruner
parent: Event-Driven
nav_order: 6
---

# Pruner

The pruner receives applied manifests via gRPC streaming. It discovers all
listable resource types in the cluster via the Kubernetes discovery API, queries
each by the labels present in OTel baggage, and diffs the cluster state against
the applied manifest set. Resources in the cluster with matching labels that are
not in the applied set are prune targets.

The pruner streams prune targets to the orderer, requesting reverse apply order.
The pruner deletes each resource as it arrives,
[impersonating](../multi-tenancy/tenant-isolation.md#impersonation) the tenant's
pruner ServiceAccount.

The pruner verifies the
[source target lease](../event-driven.md#source-target-leasing) before pruning.

The [ekbole](https://github.com/katastroma/ekbole) interface defines the gRPC
contract.

## Offboarding

Grammateus offboards a tenant by invoking the pruner with an empty manifest
stream carrying only the tenant identity in baggage — no source target labels.
The pruner queries for all resources matching that tenant label and deletes them
all, since nothing is in the intended set.

## Offboarding

Offboarding is the same flow with an empty manifest stream. Grammateus invokes
the pruner with no manifests and the tenant's identity in baggage. Since nothing
is in the intended set, all cluster resources labeled for that tenant are prune
targets and are deleted in reverse apply order.
