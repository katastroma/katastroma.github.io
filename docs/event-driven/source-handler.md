---
title: Source Handler
parent: Event-Driven
nav_order: 1
---

# Source Handler

The source handler receives events that trigger retrieval of source through
configured [Watch Targets](#watch-targets) set by the tenant.

# Source Events

When a source handler receives a source event, the source handler:

- verifies it against specific tenant secrets and signatures
- matches it against the specific tenant watch targets
- inspects the source to understand what renderer to stream the source to
- connects to the respective renderer service and streams the source to the
  renderer

# Observability

The source handler sets span attributes for each watch target (source-specific),
tenant identity, and other event metadata. These attributes are the canonical
record of what was fetched and for whom.

# Replayability

In the event of a pipeline failure, the source handler can be triggered to
replay a source retrieval and reinitiate the pipeline.

To reinitiate the pipeline, the source handler uses the RPC's event ID to query
the OTel collector and retrieve the information it needs to reconstruct each
watch target using the span attributes from the original trace.

# Replay Prevention

Before replaying each watch target, the source handler checks the
[watch target lease](../event-driven.md#watch-target-leasing) on the watch
target ConfigMap:

- If a lease is currently active (held and not stale), skip — the active
  processing supersedes this replay.
- If the replay count for this event exceeds the configured maximum, report a
  permanent failure.
