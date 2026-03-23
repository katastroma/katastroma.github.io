---
title: Cleanup
parent: Event-Driven
nav_order: 4
---

# Cleanup

A cleanup subscriber subscribes to the done subject. Every stage publishes a
done event on both success and failure:

- **On success** — deletes the completed stage's predecessor input from storage.
  Storage usage shrinks as the run progresses.
- **On failure** — deletes everything under `pipeline/{tenant}/{run-id}/`.
- **After the final stage** — revokes the cleanup credentials.

**TTL expiry** is the safety net: all objects under `pipeline/{tenant}/{run-id}/`
expire after a configured TTL regardless of cleanup subscriber activity. This
catches abandoned runs and incremental cleanup failures.
