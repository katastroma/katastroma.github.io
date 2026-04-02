---
title: Renderer
parent: Event-Driven
nav_order: 2
---

# Renderer

The renderer receives source content from the source handler via gRPC streaming
and produces Kubernetes manifests. The rendering algorithm depends on the source
type detected by the source handler (Helm chart, Kustomize overlay, or raw
YAML).

The renderer streams produced manifests to the [orderer](orderer.md).

The [keleustēs](https://github.com/katastroma/keleustes) interface defines the
gRPC contract.
