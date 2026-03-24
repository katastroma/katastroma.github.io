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

**Data flow** (gRPC streaming): [Source Handler](event-driven/source-handler.md)
→ [Renderer](event-driven/renderer.md) → [Orderer](event-driven/orderer.md) →
[Provisioner](event-driven/provisioner.md)

**Telemetry**: Each stage → OTel Collector

**Coordination/Failure Detection**: Any stage failure → OTel Collector →
[Pharos](https://github.com/katastroma/pharos) →
[Source Handler](event-driven/source-handler.md) (replay)

## Failure Recovery

When pharos receives a failed span, it follows the trace lineage to obtain the
originating source handler, connects to its service, and calls the `Replay` RPC
with the run ID (see
[Replayability](event-driven/source-handler.md#replayability)).

## Crash Detection

Pharos queries the OTel collector for runs that started but never finished
within a timeout. These are treated as failures and replayed using the same
rules.

# Scaling

Every stage is stateless. Kubernetes Service load-balances across pods. Each
stage scales independently.

# Observability

Each stage writes telemetry to the OpenTelemetry collector. Each pipeline run is
a trace. Each stage is a span. The trace ID propagates through gRPC metadata
across the entire pipeline.

## Coordination

The collector serves as both the observability backend and the coordination
layer. The OTel collector routes failed spans to pharos for
[failure recovery](#failure-recovery).

## Tenant-Facing Observability

Grafana queries the OTel collector and serves tenant-scoped dashboards showing
pipeline run status and history. Tenants see their runs in Grafana, scoped by
tenant identity. Watch target attributes on the source handler's span give
tenants visibility into which source triggered each run.
