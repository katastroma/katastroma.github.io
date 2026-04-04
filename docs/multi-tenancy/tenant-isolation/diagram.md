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
  │   ├── akrostolion implementations (label manifests with source target identity)
  │   ├── katartismos implementations (provision resources from manifests via impersonation)
  │   ├── ekbole implementations (prune resources no longer in the manifest set via impersonation)
  │   ├── pharos (pipeline coordinator)
  │   ├── OTel collector (telemetry)
  │   └── gatekeeper (admission control)
  │
  ├── CLUSTER-SCOPED RBAC (per tenant, created by grammateus)
  │   ├── ClusterRole: acme-provisioner (create, patch on *)
  │   ├── ClusterRoleBinding: acme-provisioner → SA tenant-acme/provisioner
  │   ├── ClusterRole: acme-pruner (delete on *)
  │   ├── ClusterRoleBinding: acme-pruner → SA tenant-acme/pruner
  │   ├── ClusterRole: acme-dev-provisioner (create, patch on *)
  │   ├── ClusterRoleBinding: acme-dev-provisioner → SA tenant-acme-dev/provisioner
  │   ├── ClusterRole: acme-dev-pruner (delete on *)
  │   ├── ClusterRoleBinding: acme-dev-pruner → SA tenant-acme-dev/pruner
  │   ├── ClusterRole: globex-provisioner (create, patch on *)
  │   ├── ClusterRoleBinding: globex-provisioner → SA tenant-globex/provisioner
  │   ├── ClusterRole: globex-pruner (delete on *)
  │   └── ClusterRoleBinding: globex-pruner → SA tenant-globex/pruner
  │
  ├── tenant-acme namespace (label: katastroma.org/tenant=acme, root tenant)
  │   ├── ServiceAccount: provisioner (created by grammateus)
  │   ├── ServiceAccount: pruner (created by grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: source-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── tenant-acme-dev namespace (label: katastroma.org/tenant=acme-dev, ownerRef → tenant-acme)
  │   ├── ServiceAccount: provisioner (created by grammateus)
  │   ├── ServiceAccount: pruner (created by grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: source-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── tenant-globex namespace (label: katastroma.org/tenant=globex, root tenant)
  │   ├── ServiceAccount: provisioner (created by grammateus)
  │   ├── ServiceAccount: pruner (created by grammateus)
  │   ├── Secret: webhook-secret (source handler API)
  │   ├── ConfigMap: source-target (source handler API)
  │   └── Secret: repo-credentials (source handler API)
  │
  ├── production namespace (label: katastroma.org/tenant=acme, provisioned via impersonation)
  │   └── [acme's workload pods, services, etc.]
  │
  ├── staging namespace (label: katastroma.org/tenant=acme-dev, provisioned via impersonation)
  │   └── [acme-dev's workloads]
  │
  ├── globex-app namespace (label: katastroma.org/tenant=globex, provisioned via impersonation)
  │   └── [globex's workloads]
  │
  └── GATEKEEPER POLICIES (cluster-wide)
      ├── Policy: katastroma.org/tenant label is immutable after creation
      ├── Policy: tenant SAs can only operate in namespaces matching their tenant identity label
      ├── Policy: tenant SAs must label all created resources with their tenant identity
      └── Policy: tenant SAs cannot create ClusterRoles or ClusterRoleBindings
```
