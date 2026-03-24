---
title: Security
parent: Event-Driven
nav_order: 9
has_children: true
---

# Security

The event bus and shared storage operate outside the Kubernetes API boundary —
RBAC and Gatekeeper do not intercept their operations directly. Security
controls come from three layers:

- **Kubernetes-provided** — ServiceAccount identity, Gatekeeper tenant
  exclusion, RBAC on platform Secrets, lateral movement prevention
- **Platform-enforced** — cryptographic tenant verification, STS credential
  minting restricted to the source handler, per-service publish/subscribe
  profiles
- **Solution requirements** — capabilities the event bus and storage must
  provide (token auth, ACLs, STS, TTL, audit logging)

Kubernetes controls are the foundation — they provide the identity and access
model that the event bus and storage layers build on. Tenant isolation at the
Kubernetes API level is covered in detail under
[Tenant Isolation](../multi-tenancy/tenant-isolation.md).
