---
title: Architecture
nav_order: 1
---

# Architecture

GitOps has 5 operations:

1. **Listen** - wait for events
   - Multiple event-listeners running in the system - each for different source
     types (GitHub, OCI, S3, etc.)
2. **Retrieve** — given source event, retrieve the source content
3. **Render** — given source content, produce Kubernetes manifests
4. **Order** — given unordered manifests, sort them into a safe apply order
5. **Provision** — given ordered manifests, apply them to the cluster
