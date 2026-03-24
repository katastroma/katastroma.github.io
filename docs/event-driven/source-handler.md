---
title: Source Handler
parent: Event-Driven
nav_order: 1
---

# Source Handler

The source handler receives events that trigger retrieval of source through
configured [Watch Targets](#watch-targets) set by the tenant.

# Registration

**TODO**

# Source Events

When a source handler receives a source event, the source handler:

- verifies it against specific tenant secrets and signatures
- matches it against the specific tenant watch targets
- inspects the source to understand what renderer to stream the source to
- connects to the respective renderer service and streams the source to the
  renderer

# Watch Targets

A watch target represents a source identity. The structure depends on the source
type — a git repo URL, ref, and path for a GitHub source handler, a container
registry URL and tag pattern for a container registry handler, a prefix for an
S3 handler, etc.

Tenants [register](#registration) watch targets with source handler APIs.

When a source event arrives, the source handler matches it against registered
watch targets to determine if a pipeline run is needed. If no watch target
matches, the event is ignored.

# Observability

The source handler sets span attributes for the watch target (source-specific),
tenant identity, and other run metadata. These attributes are the canonical
record of what was fetched and for whom.

# Replayability

In the event of a pipeline failure, the source handler can be triggered to
replay a source retrieval and reinitiate the pipeline.

To reinitiate the pipeline, the source handler uses the RPC's run ID to query
the OTel collector and retrieve the information it needs to reconstruct the
watch target using the span attributes from the original trace.

# Replay Prevention

Before starting the source retrieval, the source handler queries the OTel
collector to check:

- Is there already an in-flight run for this watch target? If yes, skip — the
  newer run supersedes.
- How many times has this run been replayed? If over the limit, report a
  permanent failure.
