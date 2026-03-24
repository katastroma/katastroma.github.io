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

```
Source Handler → Renderer → Orderer → Provisioner  (data, gRPC streaming)
     ↓              ↓          ↓           ↓
                 OTel Collector                     (telemetry)
     ↓              ↓          ↓           ↓
                   Pharos                           (status reporting)
```

Each stage writes telemetry to the OpenTelemetry collector. These are separate
concerns from the data flow.

## Data Flow

Each pipeline stage defines a gRPC streaming interface. The caller streams data
to the callee, the callee processes it and streams the result to the next stage.
Each call is a handoff — the caller moves on after streaming.

1. Source handler receives a source event, verifies it, matches against watch
   targets
2. Source handler fetches source, inspects it, determines renderer type and
   ordering method
3. Source handler streams source to the appropriate renderer
4. Renderer produces manifests, streams to the appropriate orderer
5. Orderer sorts manifests, streams to the provisioner
6. Provisioner applies ordered manifests to the cluster via impersonation

## Failure Recovery

When pharos receives a failed span, it follows the trace lineage to the obtain
the originating source handler, connects to its service, and calls the `Replay`
RPC with the run ID (see [Replayability](#source-handler.md#Replayability)).

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
layer. The OTel collector routes failed spans to pharos for replay coordination
(see [Replaybility](event-driven/source-handler.md#Replayability)).

## Tenant-Facing Observability

Grafana queries the OTel collector and serves tenant-scoped dashboards showing
pipeline run status and history. Tenants see their runs in Grafana, scoped by
tenant identity. Watch target attributes on the source handler's span give
tenants visibility into which source triggered each run.
