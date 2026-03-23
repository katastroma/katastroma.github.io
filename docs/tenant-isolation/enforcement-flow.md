---
title: Enforcement Flow
parent: Tenant Isolation
nav_order: 6
---

# Example Enforcement Flow

1. Grammateus onboards tenant ACME → creates tenant-acme namespace,
   acme-deployer SA with ClusterRole. ACME registers watch targets and
   credentials through the source handler APIs.
   - Gatekeeper's cluster-wide policy automatically enforces the acme-\* prefix.
2. ACME triggers webhook through source modification → source handler API
   receives event → pipeline runs → resources applied and labeled
3. GLOBEX's deployer tries to create resources in acme-prod → Gatekeeper rejects
   (globex-deployer prefix doesn't match acme-\*)
