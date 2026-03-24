---
title: Authentication
parent: Multi-Tenancy
nav_order: 5
---

# Authentication

**Open issue:** Auth architecture needs its own detailed design. Key
considerations:

- Initial tenant onboarding sets up tenant auth (IdP integration, tokens, etc.)
- Parent tenant auth can create child tenants (sub-tenant scoping)
- All platform API operations (onboarding, offboarding, child tenant creation,
  queries) are authenticated
- The auth mechanism determines who can operate on which tenant namespaces

## Authorization

Authorization determines what a tenant can do and see. This includes:

- Which tenant namespaces a caller can operate on
- Parent-to-child tenant resource visibility (a parent tenant can query child
  tenant resources, scoped by the namespace hierarchy)
- Source handler API access — registering watch targets and credentials for a
  tenant requires authorization as that tenant

## Queries

Tenant resource queryability is handled through authenticated APIs.
Authorization determines what a tenant can query:

- Resources provisioned in the cluster for a given tenant
- Namespace hierarchy and child tenants

Resource queries are backed by authorization — no separate state or inventory
tracking.
