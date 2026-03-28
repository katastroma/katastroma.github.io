---
title: Event-Driven
nav_order: 1
has_children: true
---

[pharos](https://github.com/katastroma/pharos) is the pipeline coordinator. It
embeds an OTLP receiver. The OTel collector routes failed spans to pharos.

# Pipeline

The platform decomposes GitOps into discrete stages connected by gRPC streaming.
Each stage receives data from the previous stage, does its work, and streams the
result to the next. Data flows forward only — no stage calls backward.

**Data flow** (gRPC streaming): Source event →
[Source Handler](event-driven/source-handler.md) →
[Renderer](event-driven/renderer.md) → [Orderer](event-driven/orderer.md) →
[Provisioner](event-driven/provisioner.md)

**Telemetry**: Each stage → OTel Collector

**Coordination/Failure Detection**: Any stage failure → OTel Collector →
[Pharos](https://github.com/katastroma/pharos) →
[Source Handler](event-driven/source-handler.md) (replay)

## Scaling

Every stage is stateless. Kubernetes Service load-balances across pods. Each
stage scales independently.

## Observability

Each stage writes telemetry to the OpenTelemetry collector. Each event is a
trace. Each stage is a span. The trace ID propagates through gRPC metadata
across all stages.

### Tenant-Facing Observability

## Registration

Tenants register [watch targets](#watch-targets) and any required credentials
with source handler APIs. Registration stores these as Kubernetes resources in
the tenant namespace — ConfigMaps for watch targets, Secrets for credentials.
What credentials are required (if any) depends on the source type and whether
the source is private.

Grafana queries the OTel collector and serves tenant-scoped dashboards showing
event status and history. Tenants see their events in Grafana, scoped by tenant
identity. Watch target attributes on the source handler's span give tenants
visibility into which source triggered each event.

## Watch Targets

A watch target represents a source identity. The structure depends on the source
type — a git repo URL, ref, and path for a GitHub source handler, a container
registry URL and tag pattern for a container registry handler, a prefix for an
S3 handler, etc.

Tenants [register](#registration) watch targets with source handler APIs.

When a source event arrives, the source handler matches it against registered
watch targets to determine if processing is needed. If no watch target
matches, the event is ignored.

## Events and Watch Target Processing

An event is the top-level trigger — a webhook push, a manual retrieve, or a
replay. Each event has an event ID (the OTel trace ID) and processes one or more
watch targets. Each watch target is processed independently as a child span of
the event.

The event entrypoint (webhook handler, Retrieve RPC, or Replay RPC) creates the
OTel tracer, starts the root event span, and passes both the tracer and the
context to the watch target handler. The handler creates a child span scoped to
that specific watch target.

## Watch Target Leasing

Each watch target's Kubernetes ConfigMap carries lease annotations that track
which processing instance currently owns the watch target: a watch target lease
ID (`katastroma.org/watch-target-lease-id`), a timestamp
(`katastroma.org/watch-target-lease-started`), and a replay count
(`katastroma.org/watch-target-lease-replay-count`).

The watch target lease ID is the OTel span ID of the watch target processor's
span — unique per watch target per event. The event ID (trace ID) and the watch
target lease ID (span ID) are decoupled: the event groups all watch targets
processed together, while the lease identifies the specific processing instance
for a single watch target.

Each watch target processor acquires the lease at the start of processing. New
webhook-triggered events always acquire the lease because they represent the
latest source state. Replays only proceed if no lease is currently active.

Before streaming to the renderer, each watch target processor verifies it still
holds the lease. If a newer processor has acquired the lease, the current
processor is abandoned — clone work is discarded but stale data never reaches
downstream stages. The [provisioner](event-driven/provisioner.md) performs the
same check before applying manifests.

## Failure Recovery

When pharos receives a failed span, it follows the trace lineage to obtain the
originating source handler, connects to its service, and calls the `Replay` RPC
with the event ID (see
[Replayability](event-driven/source-handler.md#replayability)).

## Crash Detection

Pharos queries the OTel collector for events that started but never finished
within a timeout. These are treated as failures and replayed using the same
[replayability rules](event-driven/source-handler.md#replayability).
