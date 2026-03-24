---
title: Observability
parent: Pipeline
nav_order: 3
---

# Observability

Each stage writes telemetry to the OpenTelemetry collector. Each pipeline run is
a trace. Each stage is a span. The trace ID propagates through gRPC metadata
across the entire pipeline.

## Tenant-Facing Observability

Grafana queries the OTel collector and serves tenant-scoped dashboards showing
pipeline run status and history. Tenants see their runs in Grafana, scoped by
tenant identity.

## Pipeline State

Pharos queries the OTel collector for pipeline run state — which runs are
in-flight, which completed, which failed. The collector is the shared state
between pharos (coordination) and Grafana (tenant observability).
