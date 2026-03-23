---
title: Deployment
nav_order: 4
---

# Deployment

The platform is deployed as Helm charts via
[phortion](https://github.com/katastroma/phortion):

- **Epibathra** — tenant management stack (tenant API server, tenant API
  frontend, gatekeeper, auth/IdP)
- **Prymna** — source event handler stack (source handler APIs, renderers,
  orderers, and provisioners)
