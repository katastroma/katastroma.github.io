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
result to the next. Each interface exposes both bidirectional streaming and client-streaming RPCs — consumers choose the pattern that fits.

**Data flow** (gRPC streaming): Source event →
[Source Handler](event-driven/source-handler.md) →
[Renderer](event-driven/renderer.md) → [Orderer](event-driven/orderer.md) →
[Labeler](event-driven/labeler.md) →
[Provisioner](event-driven/provisioner.md) →
[Pruner](event-driven/pruner.md)

**Telemetry**: Each stage → OTel Collector

**Coordination/Failure Detection**: Any stage failure → OTel Collector →
[Pharos](https://github.com/katastroma/pharos) →
[Source Handler](event-driven/source-handler.md) (replay)

## Scaling

Every stage is stateless. Kubernetes Service load-balances across pods. Each
stage scales independently.

## Observability

Each stage writes telemetry to the OpenTelemetry collector. Each event is a
trace. Each stage is a span. OTel trace context and baggage propagate across all
stages.

## Registration

Tenants register [source targets](#source-targets) and any required credentials
with source handler APIs. Registration stores these as Kubernetes resources in
the tenant namespace — ConfigMaps for source targets, Secrets for credentials.
Credential Secrets are in the tenant namespace and protected by the same
[label-based isolation](multi-tenancy/tenant-isolation.md) as all tenant
resources. What credentials are required (if any) depends on the source type and
whether the source is private.

Grafana queries the OTel collector and serves tenant-scoped dashboards showing
event status and history. Tenants see their events in Grafana, scoped by tenant
identity. Source target attributes on each source target's child span give
tenants visibility into which source triggered each event.

## Source Targets

A source target represents a source identity. The structure depends on the
source type — a git repo URL, ref, and path for a GitHub source handler, a
container registry URL and tag pattern for a container registry handler, a
prefix for an S3 handler, etc.

Tenants [register](#registration) source targets with source handler APIs.

When a source event arrives, the source handler matches it against registered
source targets to determine if processing is needed. If no source target
matches, the event is ignored.

## Events and Source Target Processing

An event is the top-level trigger — a webhook push, a manual retrieve, or a
replay. Each event has an event ID (the OTel trace ID) and processes one or more
source targets. Each source target is processed independently as a child span of
the event.

The event entrypoint (webhook handler, Retrieve RPC, or Replay RPC) creates the
OTel tracer, starts the root event span, and passes both the tracer and the
context to the source target handler. The handler creates a child span scoped to
that specific source target.

## Source Target Leasing

Each source target's Kubernetes ConfigMap carries lease annotations that track
which processing instance currently owns the source target: a lease ID
(`katastroma.org/lease-id`), a timestamp (`katastroma.org/lease-started`), and a
replay count (`katastroma.org/lease-replay-count`).

The lease ID is the OTel span ID of the source target processor's span — unique
per source target per event. The event ID (trace ID) and the source target lease
ID (span ID) are decoupled: the event groups all source targets processed
together, while the lease identifies the specific processing instance for a
single source target.

Each source target processor acquires the lease at the start of processing. New
webhook-triggered events always acquire the lease because they represent the
latest source state. Replays only proceed if no lease is currently active.

Before streaming to the renderer, each source target processor verifies it still
holds the lease. If a newer processor has acquired the lease, the current
processor is abandoned — clone work is discarded but stale data never reaches
downstream stages. The [provisioner](event-driven/provisioner.md) and
[pruner](event-driven/pruner.md) perform the same check before applying or
removing resources.

## Failure Recovery

When pharos receives a failed span, it follows the trace lineage to obtain the
originating source handler, connects to its service, and calls the `Replay` RPC
with the event ID (see
[Replayability](event-driven/source-handler.md#replayability)).

## Crash Detection

Pharos queries the OTel collector for events that started but never finished
within a timeout. These are treated as failures and replayed using the same
[replayability rules](event-driven/source-handler.md#replayability).
