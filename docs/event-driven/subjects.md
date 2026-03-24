---
title: Subjects
parent: Event-Driven
nav_order: 1
---

# Subjects

Each stage announces what it produced. The subject describes the artifact and
encodes routing for the next consumer. The producing stage does not know or name
the consumer.

```
{domain}.source.ready.{renderer_type}
{domain}.render.ready.{ordering_method}
{domain}.order.ready
{domain}.pipeline.done
```

Subjects encode the artifact type and routing — not tenant identity. Tenant
identity is in the event payload. No pipeline service routes by tenant.

The source handler determines the renderer type (from source inspection) and
ordering method. The ordering method selects which ordering algorithm to apply
to the rendered manifests. Multiple ordering implementations can run
simultaneously, each subscribing to its own subject. The Kubernetes ecosystem
provides several ordering algorithms (e.g., Helm's `InstallOrder`, the
`cli-utils` GVK sort) and the subject model allows sources to target whichever
is appropriate.

Each downstream stage reads the routing value it needs from the event payload
and uses it in its outgoing subject.

Each implementation subscribes to the subject that matches what it consumes. A
helm renderer subscribes to `{domain}.source.ready.helm`. A helm-sort orderer
subscribes to `{domain}.render.ready.helm-sort`. Multiple instances of the same
implementation form a consumer group — the event bus distributes across them.
Adding a new implementation is deploying a service and subscribing to a new
subject. No existing services change.

Every stage also publishes to the done subject with its outcome. The cleanup
subscriber and observability layer consume done events.
