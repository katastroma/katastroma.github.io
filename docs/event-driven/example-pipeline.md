---
title: Example Pipeline
parent: Event-Driven
nav_order: 10
---

# Example Pipeline

1. Source event → source handler receives it
2. Source handler verifies webhook signature → identifies tenant
3. Source handler matches event against watch targets — skips if no match
4. Source handler retrieves source, inspects it, determines renderer type and
   ordering method
5. Source handler mints storage credentials, writes source to
   `pipeline/{tenant}/{run-id}/source`, publishes "source ready", publishes done
6. Renderer reads source, renders manifests, writes to
   `pipeline/{tenant}/{run-id}/manifests-unordered`, publishes "render ready",
   publishes done → cleanup deletes source
7. Orderer reads unordered manifests, orders them, writes to
   `pipeline/{tenant}/{run-id}/manifests-ordered`, publishes "order ready",
   publishes done → cleanup deletes unordered manifests
8. Provisioner reads ordered manifests, applies to cluster via impersonation,
   publishes done → cleanup deletes ordered manifests and revokes cleanup
   credentials
