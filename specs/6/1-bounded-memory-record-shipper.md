---
status: draft
---

# Bounded-memory record shipper — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule stalls the collector for a fixed interval at an exact
record boundary, resumes it at a byte-per-read rate, closes the connection
mid-record, resets the connection after acknowledging a known position, and
sends SIGTERM while a submission is outstanding. It grows the log file
during every stall.

Checks do not inspect private functions or require a named I/O interface.
They observe the received byte stream, the acknowledgement sequence, the
resume position after restart, resident memory over time, and telemetry.
