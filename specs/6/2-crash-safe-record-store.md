---
status: draft
---

# Crash-safe record store — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

The failure schedule kills the process and the backing device at exact record
boundaries, drops unflushed data on resume, returns one I/O error to a flush
call and then resumes normal behavior, truncates the tail of a file, and flips
bytes inside a stored record, then restarts the store. Verification replays
the full client history against what the store serves.

Checks do not inspect private functions or require a named on-disk format.
They observe acknowledgements, served records, recovery duration, and
reported errors.
