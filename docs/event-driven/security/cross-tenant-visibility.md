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
2. **Per-event storage credentials** — a service processing tenant acme's event
   cannot use those credentials to access tenant globex's storage.
3. **Read/write credentials expire** — cleanup credentials are delete-only.
4. **Per-run credential independence** — credentials from one run cannot access
   another run's storage.
5. **Audit logging** — all event bus and storage activity is logged by
   credential.
