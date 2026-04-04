---
title: Home
layout: home
nav_order: 0
---

# Katastroma

Source event-driven platform for Kubernetes. Multi-tenant support and tenant
isolation through Kubernetes-native RBAC, impersonation, and admission control.

# Architecture

GitOps has 5 operations:

1. **Listen** - wait for events
   - Multiple source handlers running in the system - each for different source
     types (GitHub, OCI, S3, etc.)
2. **Retrieve** — given source event, retrieve the source content
3. **Render** — given source content, produce Kubernetes manifests
4. **Order** — given unordered manifests, sort them into a safe apply order
5. **Provision** — given ordered manifests, apply them to the cluster

# Components

Interface repos define client-facing gRPC APIs. Clients depend on the interface
without importing the implementation. Implementation repos consume the interface
and provide the concrete service.

| Component                                                | Role                                   |
| -------------------------------------------------------- | -------------------------------------- |
| [grammateus](https://github.com/katastroma/grammateus)   | Tenant management API server           |
| [prora](https://github.com/katastroma/prora)             | Tenant self-service frontend           |
| [naukleros](https://github.com/katastroma/naukleros)     | Source handler interface               |
| [phortizo](https://github.com/katastroma/phortizo)       | Source handler implementation (GitHub) |
| [keleustēs](https://github.com/katastroma/keleustes)     | Renderer interface                     |
| [orpheus](https://github.com/katastroma/orpheus)         | Renderer implementation                |
| [diataxis](https://github.com/katastroma/diataxis)       | Orderer interface                      |
| [stolarches](https://github.com/katastroma/stolarches)   | Orderer implementation                 |
| [akrostolion](https://github.com/katastroma/akrostolion) | Labeler interface                      |
| [parasemon](https://github.com/katastroma/parasemon)     | Labeler implementation                 |
| [katartismos](https://github.com/katastroma/katartismos) | Provisioner interface                  |
| [histia](https://github.com/katastroma/histia)           | Provisioner implementation             |
| [ekbole](https://github.com/katastroma/ekbole)           | Pruner interface                       |
| [ekboleus](https://github.com/katastroma/ekboleus)       | Pruner implementation                  |
| [pharos](https://github.com/katastroma/pharos)           | Pipeline coordinator                   |

# Deployment

All platform resources run in the **platform namespace** (default `katastroma`,
configurable). Tenant resources are in namespaces labeled with the tenant
identity. The platform namespace is where all platform services, Gatekeeper, and
platform Secrets reside.

The platform is deployed as Helm charts via
[phortion](https://github.com/katastroma/phortion):

- **Epibathra** — tenant management stack (tenant API server, tenant API
  frontend, Grafana, gatekeeper, auth/IdP)
- **Prymna** — pipeline stack (pipeline coordinator, OTel collector, source
  handler APIs, renderers, orderers, and provisioners)
