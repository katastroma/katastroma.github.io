---
title: Orderer
parent: Event-Driven
nav_order: 3
---

# Orderer

The orderer receives unordered Kubernetes manifests from the renderer via gRPC
streaming and sorts them into a safe apply order.

Kubernetes resources can depend on other resources — a resource in a Namespace
requires the Namespace to exist, a custom resource requires its
CustomResourceDefinition, a RoleBinding references a ServiceAccount. Applying
resources out of order fails when a dependent resource arrives before its
prerequisite.

The orderer determines these dependencies and sequences manifests so
prerequisites are applied first. It streams the ordered manifests to the
[provisioner](provisioner.md).

The [diataxis](https://github.com/katastroma/diataxis) interface defines the
gRPC contract.
