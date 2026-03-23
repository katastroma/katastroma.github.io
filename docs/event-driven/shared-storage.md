---
title: Shared Storage
parent: Event-Driven
nav_order: 3
---

# Shared Storage

Ephemeral scratch space. Artifacts exist only for the duration of a pipeline
run:

```
pipeline/{tenant}/{run-id}/source
pipeline/{tenant}/{run-id}/manifests-unordered
pipeline/{tenant}/{run-id}/manifests-ordered
```

Each stage reads its predecessor's output, writes its own, and the predecessor's
can be deleted. Only one artifact per run needs to exist at a time.
