---
title: Coordination
parent: Pipeline
nav_order: 2
---

# Coordination

Pharos is the pipeline coordinator. Each stage reports to pharos when it starts
and finishes a run. Pharos uses this to detect failures and dispatch replays.

## Status Reporting

Each stage makes a gRPC call to pharos with: run ID, tenant, watch target,
stage name, and outcome (started, succeeded, or failed).

## Failure Recovery

When a stage reports failure, pharos dispatches a replay to the source handler:
re-fetch the source using the same parameters as the original event and restart
the pipeline. Before dispatching, pharos queries the OTel collector to check:

- Is there already an in-flight run for this tenant + watch target? If yes,
  don't replay — the newer run supersedes.
- How many times has this run been replayed? If over the limit, record a
  permanent failure.

## Crash Detection

If a stage crashes, it never reports back. Pharos queries the OTel collector for
runs that started but never finished within a timeout. These are treated as
failures and replayed using the same rules.

## Replay Rules

- A replay re-fetches the source using the exact parameters from the original
  event. The source tells the pipeline what to fetch — the pipeline does not
  take assumptions about what is latest.
- A replay is a new run with a new run ID.
- A replay is not dispatched if a newer run for the same tenant + watch target
  is already in-flight.
