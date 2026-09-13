---
status: draft
---

# Steady-state request service — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule opens a cohort of long-lived workspaces before anything
else and keeps them open across the entire run, reading their entries
throughout. At a named workspace identity it starts the peak: a flood of
short-lived workspaces with large entries that opens, fills, reads, and
closes until live data reaches its stated peak, then ends at a second named
identity. The steady phase that follows uses small entries only. Every read
is verified byte for byte against what was stored, and the schedule sends
SIGTERM at a named identity while requests are in flight and expects a clean
drain.

Checks do not inspect private functions or require a named memory strategy.
They observe responses, resident memory over time, the live data implied by
the request history, and the response-time distribution per phase.
