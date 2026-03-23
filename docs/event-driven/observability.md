---
title: Observability
parent: Event-Driven
nav_order: 7
---

# Observability

The event bus is the source of truth for pipeline run history. Every stage
transition is a persisted event with full metadata. The event bus retains
messages for a configurable duration (days, months, whatever is needed).

Wildcard subscriptions allow subscribing to all events for a given tenant in
real time, enabling the tenant-facing frontend to show live pipeline progress
and historical run queries without a separate database.
