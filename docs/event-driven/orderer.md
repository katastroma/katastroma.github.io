---
title: Orderer
parent: Event-Driven
nav_order: 3
---

# Orderer

The orderer receives unordered Kubernetes manifests from the renderer via gRPC
streaming and sorts them into a safe apply order. The sorting algorithm
determines resource dependencies and sequences resources so that prerequisites
are applied before dependents.

Orderer implementations use sorting libraries directly — no shell execution. The
[diataxis](https://github.com/katastroma/diataxis) interface defines the gRPC
contract.
