---
title: Pipeline
nav_order: 1
has_children: true
---

# Pipeline

The platform decomposes GitOps into discrete stages connected by gRPC streaming.
Each stage receives data from the previous stage, does its work, and streams the
result to the next. Data flows forward only — no stage calls backward.

```
Source Handler → Renderer → Orderer → Provisioner  (data, gRPC streaming)
     ↓              ↓          ↓           ↓
                   Pharos                           (status reporting)
     ↓              ↓          ↓           ↓
                 OTel Collector                     (telemetry)
```

Each stage also reports its status to pharos (pipeline coordinator) and writes
telemetry to the OpenTelemetry collector. These are separate concerns from the
data flow.

## Watch Targets

A watch target represents a tracked source location. The structure depends on
the source type — a git ref and path for a GitHub source handler, a tag pattern
for a container registry handler, a prefix for an S3 handler, etc. Tenants
register watch targets with source handler APIs during onboarding. When a source
event arrives, the source handler matches it against registered watch targets to
determine if a pipeline run is needed. If no watch target matches, the event is
ignored.
