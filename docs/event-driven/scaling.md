---
title: Scaling
parent: Event-Driven
nav_order: 8
---

# Scaling

Every service in the pipeline is stateless — it reads from storage, does its
work, writes to storage, publishes events. No local state, no session affinity,
no coordination between instances. Each service scales independently:

- Rendering slow? Scale renderers. Orderers and provisioners are unaffected.
- Ordering trivial? Run one instance.
- Provisioning bottlenecked on K8s API throughput? Scale provisioners up to what
  the API server can handle.

The event bus distributes events across consumer group members automatically.
The shared storage scales independently of the event bus. No architectural
ceiling on horizontal scaling.

## Concurrent Runs

Multiple pipeline runs for the same tenant can be in-flight simultaneously. Each
run has its own run ID, storage path, and credentials — runs do not interact.
Per-tenant concurrency limits are enforced at the source handler (see
[Quotas](quotas.md)), not by the pipeline stages themselves.
