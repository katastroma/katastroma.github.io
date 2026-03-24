---
title: Event Payload
parent: Event-Driven
nav_order: 2
---

# Event Payload

- **Run ID** — unique identifier for this pipeline run
- **Tenant** — tenant identity
- **Stage** — which pipeline stage produced this event
- **Timestamp** — when the event was produced
- **Outcome** — success or failure of the producing stage
- **Storage key** — pointer to the current stage's output in shared storage
- **Storage credentials** — short-lived read/write, scoped to
  `pipeline/{tenant}/{run-id}/*`. Expire by TTL — no explicit revocation needed.
- **Cleanup credentials** — long-lived delete-only, same scope. Revoked
  explicitly by the cleanup subscriber after the final stage.
- **Pipeline routing** — renderer type and ordering method
- **Service account** — the tenant's deployer SA identity for impersonation
- **Labels** — tenant and platform labels for provisioned resources
