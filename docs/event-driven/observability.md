---
title: Observability
parent: Event-Driven
nav_order: 7
---

# Observability

The event bus is the source of truth for pipeline run history. Every stage
transition is a persisted event with full metadata. The event bus retains
messages for a configurable duration (days, months, whatever is needed).

## Real-Time Pipeline Status

Pharos is the observability service. It subscribes to all pipeline event
subjects, filters events by tenant identity from the payload, and serves
tenant-scoped pipeline status to frontend clients over WebSocket.

The frontend (prora) authenticates with grammateus and receives a token. It
opens a WebSocket connection to pharos with that token. Pharos validates the
token, extracts the tenant identity, and streams only that tenant's pipeline
events to the connection.

Pipeline services do not serve frontend connections. Each service has one job —
process its stage's events. Pharos is the only service that bridges the event
bus to the frontend.

## Historical Queries

Historical pipeline run data is queried from the event bus's persisted message
streams. Pharos serves historical queries using the same tenant-scoped
authorization — a tenant can only query its own runs.
