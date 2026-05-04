---
title: Labeler
parent: Event-Driven
nav_order: 4
---

# Labeler

The labeler receives ordered manifests from the orderer and stamps each with
labels derived from OTel baggage. Applicable baggage entries are applied as
labels on each manifest.

**Open design point:** what constitutes an "applicable" baggage entry — the
criteria by which the labeler selects which baggage keys to promote to labels —
is not yet defined.

The labeler streams each labeled manifest to the
[provisioner](provisioner.md) as it arrives — no buffering.

The [akrostolion](https://github.com/katastroma/akrostolion) interface defines
the gRPC contract.
