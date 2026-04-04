---
title: Labeler
parent: Event-Driven
nav_order: 4
---

# Labeler

The labeler receives ordered manifests from the orderer and stamps each with
platform labels identifying the source target. It reads the source target
identity from OTel baggage.

The labeler streams each labeled manifest to the
[provisioner](provisioner.md) as it arrives — no buffering.

The [akrostolion](https://github.com/katastroma/akrostolion) interface defines
the gRPC contract.
