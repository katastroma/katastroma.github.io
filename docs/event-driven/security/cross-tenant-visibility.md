---
title: Cross-Tenant Visibility
parent: Security
grand_parent: Event-Driven
nav_order: 4
---

# Cross-Tenant Visibility

## What Cannot Be Per-Tenant

Source handlers, renderers, orderers, and provisioners process events for all
tenants within their stage. A renderer subscribed to
`{domain}.source.ready.helm.*` receives events for every tenant. Running a
separate instance per tenant does not scale to thousands of tenants.

A compromised service can observe event metadata for all tenants within its
stage. It cannot observe events outside its stage.

## Mitigations

1. **Stage-scoped permissions** — blast radius is one stage, not the full
   pipeline.
2. **Per-event, per-run storage credentials** — scoped to
   `pipeline/{tenant}/{run-id}/*`. A service processing tenant acme's event
   cannot access tenant globex's storage, and credentials from one run cannot
   access another. See [Platform-Enforced Controls](platform-controls.md) for
   the credential minting model.
3. **Read/write credentials expire** — cleanup credentials are delete-only.
4. **Audit logging** — all event bus and storage activity is logged by
   credential.
