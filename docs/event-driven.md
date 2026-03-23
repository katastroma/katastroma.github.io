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
