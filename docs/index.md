---
title: Home
layout: home
nav_order: 0
---

# Katastroma

Source event-driven platform for Kubernetes. Multi-cluster and multi-tenant
support and tenant isolation through Kubernetes-native RBAC, impersonation, and
admission control.

# Architecture

GitOps has 5 operations:

1. **Listen** - wait for events
   - Multiple event-listeners running in the system - each for different source
     types (GitHub, OCI, S3, etc.)
2. **Retrieve** — given source event, retrieve the source content
3. **Render** — given source content, produce Kubernetes manifests
4. **Order** — given unordered manifests, sort them into a safe apply order
5. **Provision** — given ordered manifests, apply them to the cluster

# Components

| Component                                                | Role                         |
| -------------------------------------------------------- | ---------------------------- |
| [grammateus](https://github.com/katastroma/grammateus)   | Tenant management API server |
| [prora](https://github.com/katastroma/prora)             | Tenant self-service frontend |
| [phortizo](https://github.com/katastroma/phortizo)       | GitHub Source Event Listener |
| [keleustēs](https://github.com/katastroma/keleustes)     | Renderer interface           |
| [orpheus](https://github.com/katastroma/orpheus)         | Renderer implementation      |
| [diataxis](https://github.com/katastroma/diataxis)       | Orderer interface            |
| [stolarches](https://github.com/katastroma/stolarches)   | Orderer implementation       |
| [katartismos](https://github.com/katastroma/katartismos) | Provisioner interface        |
| [histia](https://github.com/katastroma/histia)           | Provisioner implementation   |

# Deployment

The platform is deployed as Helm charts via
[phortion](https://github.com/katastroma/phortion):

- **Epibathra** — tenant management stack (tenant API server, tenant API
  frontend, gatekeeper, auth/IdP)
- **Prymna** — source event handler stack (source handler APIs, renderers,
  orderers, and provisioners)
