---
title: Provisioner
parent: Event-Driven
nav_order: 4
---

# Provisioner

The provisioner receives ordered manifests from the orderer via gRPC streaming
and applies them to the cluster. It
[impersonates](../multi-tenancy/tenant-isolation.md#impersonation) the tenant's
deployer ServiceAccount so that Kubernetes evaluates authorization against the
tenant's permissions, not the platform's.

Before applying manifests, the provisioner verifies the
[source target lease](../event-driven.md#source-target-leasing).

The [katartismos](https://github.com/katastroma/katartismos) interface defines
the gRPC contract.
