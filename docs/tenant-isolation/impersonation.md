---
title: Impersonation
parent: Tenant Isolation
nav_order: 1
---

# Impersonation

Histia provisions resources using native Kubernetes impersonation
(Impersonate-User HTTP headers). When applying manifests for a tenant, histia
impersonates the tenant's deployer SA. The impersonated SA has a ClusterRole
with `create`, `patch`, and `delete` on all resources, but Gatekeeper constrains
where those permissions apply based on namespace prefix. Kubernetes RBAC
escalation prevention ensures tenants cannot grant themselves broader
permissions than their SA has.
