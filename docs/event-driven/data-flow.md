---
title: Data Flow
parent: Pipeline
nav_order: 1
---

# Data Flow

Each pipeline stage defines a gRPC streaming interface. The caller streams data
to the callee, the callee processes it and streams the result to the next stage.
Each call is a handoff — the caller moves on after streaming.

1. Source handler receives a webhook, verifies it, matches against watch targets
2. Source handler fetches source, inspects it, determines renderer type and
   ordering method
3. Source handler streams source to the appropriate renderer
4. Renderer produces manifests, streams to the appropriate orderer
5. Orderer sorts manifests, streams to the provisioner
6. Provisioner applies ordered manifests to the cluster via impersonation

## Routing

The source handler determines which renderer and orderer to use based on the
source content. Each implementation runs behind its own Kubernetes Service.

## Scaling

Every stage is stateless. Kubernetes Service load-balances across pods. Each
stage scales independently.
