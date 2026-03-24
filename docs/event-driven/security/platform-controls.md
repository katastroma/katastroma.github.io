---
title: Platform-Enforced Controls
parent: Security
grand_parent: Event-Driven
nav_order: 2
---

# Platform-Enforced Controls

- **Tenant identity is cryptographically verified once** — the source handler
  validates the webhook HMAC signature against the tenant's stored secret. All
  downstream security decisions derive from this verification.
- **Replay prevention is a source handler responsibility** — HMAC alone does not
  prevent an attacker from re-sending a previously signed payload. Source
  handlers must reject replayed events via timestamp validation or idempotency
  checks. This is solely a source handler concern because the source handler is
  the only service that receives external payloads. All downstream communication
  is internal — services publish to the event bus using authenticated platform
  SAs, not external actors. Duplicate internal events from a compromised SA are
  an SA compromise scenario, not a replay attack.
- **Storage credential minting** — the source handler is the only service
  authorized to call the storage STS. It mints two credential sets per run, both
  scoped to `pipeline/{tenant}/{run-id}/*`: short-lived read/write for pipeline
  stages, and long-lived delete-only for cleanup. Downstream stages use
  credentials from the event payload — they never mint their own. If any other
  service attempts to call STS, the request is rejected.
- **Per-service publish/subscribe profiles** — the platform defines what each SA
  can publish and subscribe to. The event bus enforces these as per-connection
  ACLs tied to the SA's TokenReview-authenticated identity (see
  [Solution Requirements](solution-requirements.md)):
  - Source handlers: publish to source.ready and done
  - Renderers: subscribe to source.ready, publish to render.ready and done
  - Orderers: subscribe to render.ready, publish to order.ready and done
  - Provisioners: subscribe to order.ready, publish to done
  - Cleanup subscriber: subscribe to done
