---
title: Retries
parent: Event-Driven
nav_order: 6
---

# Retries

The event bus redelivers events on failure with configurable backoff. If the
service crashed, the event is redelivered to another instance in the consumer
group. After a configurable maximum, the event moves to a dead letter subject
and the stage publishes a done event with a failure outcome.
