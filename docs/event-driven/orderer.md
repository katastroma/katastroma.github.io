---
title: Orderer
parent: Event-Driven
nav_order: 3
---

# Orderer

The orderer receives Kubernetes manifests via gRPC streaming and sorts them by
dependency.

Kubernetes resources can depend on other resources — a resource in a Namespace
requires the Namespace to exist, a custom resource requires its
CustomResourceDefinition, a RoleBinding references a ServiceAccount. Applying
resources out of order fails when a dependent resource arrives before its
prerequisite.

The orderer determines these dependencies and sequences manifests accordingly.
Callers specify the desired sort direction to return — apply order
(prerequisites first) or reverse apply order (dependents first, for pruning).

The [diataxis](https://github.com/katastroma/diataxis) interface defines the
gRPC contract.
