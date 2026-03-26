---
title: Provisioner
parent: Event-Driven
nav_order: 4
---

# Provisioner

Provisioner applies ordered manifests to the cluster via impersonation.

## Stale Run Prevention

Before applying manifests, the provisioner checks the
[run ownership](../event-driven.md#run-ownership) lease on the watch target
ConfigMap. If the current run no longer holds the lease, the provisioner
abandons the apply. This is defense in depth — the source handler performs the
same check before streaming, but timing gaps between stages can allow a stale
run to reach the provisioner.
