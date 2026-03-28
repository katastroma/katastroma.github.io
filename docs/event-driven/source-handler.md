---
title: Source Handler
parent: Event-Driven
nav_order: 1
---

# Source Handler

The source handler receives source events and matches them against registered
[source targets](../event-driven.md#source-targets) to determine which sources
to retrieve and process.

# Source Events

When a source handler receives a source event, it processes the event through
the following pipeline:

1. Verify the event against tenant-stored secrets
2. Match the event against registered source targets
3. For each matched source target:
   - Acquire the [source target lease](../event-driven.md#source-target-leasing)
   - Resolve tenant credentials
   - Retrieve the source content
   - Detect the renderer type
   - Verify the lease is still held
   - Stream the source to the renderer

# Observability

The source handler sets span attributes for each source target
(source-specific), tenant identity, and other event metadata. These attributes
are the canonical record of what was fetched and for whom.

# Replayability

In the event of a pipeline failure, the source handler can be triggered to
replay a source retrieval and reinitiate the pipeline.

To reinitiate the pipeline, the source handler uses the RPC's event ID to query
the OTel collector and retrieve the information it needs to reconstruct each
source target using the span attributes from the original trace.

# Replay Prevention

Before replaying each source target, the source handler checks the
[source target lease](../event-driven.md#source-target-leasing) on the source
target ConfigMap:

- If a lease is currently active (held and not stale), skip — the active
  processing supersedes this replay.
- If the replay count for this event exceeds the configured maximum, report a
  permanent failure.
