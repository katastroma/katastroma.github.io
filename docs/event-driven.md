---
title: Event-Driven
nav_order: 2
has_children: true
---

# Event-Driven

The platform has two infrastructure planes:

- **Event bus** — control plane. Signals between pipeline stages via
  subject-based routing and persistent message streams.
- **Shared object storage** — data plane. Carries artifacts (source content,
  manifests) between pipeline stages via tenant-scoped, run-scoped storage keys.

## Watch Targets

A watch target represents a tracked source location. The structure depends on
the source type — a git ref and path for a GitHub source handler, a tag pattern
for a container registry handler, a prefix for an S3 handler, etc. Tenants
register watch targets with source handler APIs during onboarding. When a source
event arrives, the source handler matches it against registered watch targets to
determine if a pipeline run is needed. If no watch target matches, the event is
ignored.
