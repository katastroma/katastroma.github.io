---
title: Quotas
parent: Event-Driven
nav_order: 5
---

# Quotas

A tenant's storage footprint is bounded by concurrent in-flight runs multiplied
by maximum data size per run. The source handler enforces a maximum source size
before writing to storage. TTL caps accumulation even under burst traffic.

Per-tenant concurrency limits can be enforced at the source handler — if a
tenant exceeds a maximum number of in-flight runs, new events are rejected or
queued.
