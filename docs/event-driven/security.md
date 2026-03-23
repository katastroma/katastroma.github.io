---
title: Security
parent: Event-Driven
nav_order: 9
has_children: true
---

# Security

The event bus and shared storage operate outside the Kubernetes API boundary.
Security controls come from three layers: what Kubernetes provides natively, what
the platform enforces through its own design, and what the event bus and storage
solutions must provide as capabilities.
