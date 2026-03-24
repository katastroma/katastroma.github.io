---
title: Solution Requirements
parent: Security
grand_parent: Event-Driven
nav_order: 3
---

# Event Bus and Storage Solution Requirements

- SA token authentication via TokenReview API
- Per-connection publish/subscribe ACLs
- STS with prefix-scoped temporary credentials
- STS access control restricting which SAs can mint credentials
- Credential revocation
- TTL-based object expiry
- Audit logging — all authentication, publish, subscribe, and storage access
  activity logged by credential
