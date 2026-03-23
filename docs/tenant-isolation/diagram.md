---
title: Diagram
parent: Tenant Isolation
nav_order: 7
---

# Cluster Diagram

```
CLUSTER
  ├── platform namespace
  │   ├── grammateus (tenant API server)
  │   ├── source handler APIs (receive source events, fetch source)
  │   ├── keleustes implementations (render manifests from source)
  │   ├── diataxis implementations (order manifests for safe apply)
  │   ├── katartismos implementations (provision resources from manifests via impersonation)
  │   ├── event bus (pipeline coordination)
  │   ├── shared storage (artifacts between pipeline stages)
  │   └── gatekeeper (admission control)
  │
  ├── CLUSTER-SCOPED RBAC (per tenant, created by grammateus)
  │   ├── ClusterRole: acme-deployer (create, patch, delete on *)
  │   ├── ClusterRoleBinding: acme-deployer → SA tenant-acme/acme-deployer
  │   ├── ClusterRole: acme-dev-deployer (create, patch, delete on *)
  │   ├── ClusterRoleBinding: acme-dev-deployer → SA tenant-acme-dev/acme-dev-deployer
  │   ├── ClusterRole: globex-deployer (create, patch, delete on *)
  │   └── ClusterRoleBinding: globex-deployer → SA tenant-globex/globex-deployer
  │
  ├── tenant-acme namespace (root tenant)
  │   ├── ServiceAccount: acme-deployer (grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: watch-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── tenant-acme-dev namespace (child, ownerRef → tenant-acme)
  │   ├── ServiceAccount: acme-dev-deployer (grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: watch-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── tenant-globex namespace (root tenant)
  │   ├── ServiceAccount: globex-deployer (grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: watch-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── acme-prod namespace (provisioned by histia from tenant manifests, impersonating acme-deployer)
  │   └── [acme's workload pods, services, etc.]
  │
  ├── acme-dev-staging namespace (provisioned by histia from tenant manifests, impersonating acme-dev-deployer)
  │   └── [acme-dev's workloads]
  │
  ├── globex-app namespace (provisioned by histia from tenant manifests, impersonating globex-deployer)
  │   └── [globex's workloads]
  │
  └── GATEKEEPER POLICIES (cluster-wide, not per-tenant)
      ├── Policy: SAs can only create namespaces matching their tenant prefix
      ├── Policy: resources in a namespace can only be modified by an SA whose prefix matches
      └── Policy: tenant deployer SAs cannot create ClusterRoles or ClusterRoleBindings
```
