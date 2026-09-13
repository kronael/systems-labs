---
status: draft
---

# Serverless quote aggregation — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier, never on a timer and never at random.
The failure schedule drives offered load several times the declared ceiling,
runs the provider latency schedule including a sustained slow interval and a
hard failure at named requests, destroys the environment that served a named
request so the next named request pays first-invocation latency, freezes the
environment with a named request's provider call outstanding, and holds a
named request's provider call past the handler's maximum run time.

Checks do not require a named cache or rejection mechanism. They observe
the public API, the expiry stamped on each returned quote, provider-side
request traffic, invocation and throttle counts, telemetry, and the submitted
evidence.
