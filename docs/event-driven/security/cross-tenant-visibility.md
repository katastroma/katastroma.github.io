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
`{domain}.source.ready.helm` receives events for every tenant. Running a
separate instance per tenant does not scale to thousands of tenants.

A compromised service can observe event metadata for all tenants within its
stage. It cannot observe events outside its stage.

## Event Payload Sensitivity

Event payloads must contain only what is needed for routing and storage access —
run ID, tenant identity, stage, outcome, storage key, credentials, pipeline
routing, service account, and labels. Event payloads must not contain source
content, rendered manifests, error details, or any tenant-specific data beyond
operational metadata. Tenant data flows through shared storage with per-run
scoped credentials, not through the event bus.

A compromised service within a stage can observe which tenants exist, how often
they deploy, and whether runs succeed or fail. It cannot observe what tenants
are deploying — that data is in storage, protected by per-event credentials.

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
