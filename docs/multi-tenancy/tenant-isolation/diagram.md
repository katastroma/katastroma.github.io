---
title: Diagram
parent: Tenant Isolation
grand_parent: Multi-Tenancy
nav_order: 1
---

# Cluster Diagram

```
CLUSTER
  ├── platform namespace (katastroma)
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
  ├── tenant-acme namespace (label: katastroma.io/tenant=acme, root tenant)
  │   ├── ServiceAccount: acme-deployer (created by grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: watch-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── tenant-acme-dev namespace (label: katastroma.io/tenant=acme-dev, ownerRef → tenant-acme)
  │   ├── ServiceAccount: acme-dev-deployer (created by grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: watch-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── tenant-globex namespace (label: katastroma.io/tenant=globex, root tenant)
  │   ├── ServiceAccount: globex-deployer (created by grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: watch-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── production namespace (label: katastroma.io/tenant=acme, provisioned by histia impersonating acme-deployer)
  │   └── [acme's workload pods, services, etc.]
  │
  ├── staging namespace (label: katastroma.io/tenant=acme-dev, provisioned by histia impersonating acme-dev-deployer)
  │   └── [acme-dev's workloads]
  │
  ├── globex-app namespace (label: katastroma.io/tenant=globex, provisioned by histia impersonating globex-deployer)
  │   └── [globex's workloads]
  │
  └── GATEKEEPER POLICIES (cluster-wide)
      ├── Policy: katastroma.io/tenant label is immutable after creation
      ├── Policy: deployer SAs can only operate in namespaces matching their tenant identity label
      ├── Policy: deployer SAs must label all created resources with their tenant identity
      └── Policy: deployer SAs cannot create ClusterRoles or ClusterRoleBindings
```
