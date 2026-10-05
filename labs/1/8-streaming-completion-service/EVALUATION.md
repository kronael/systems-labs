> Spoilers. Open only when stuck.

# Streaming completion service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
The schedule drives the declared prompt distribution and then:

- when a named request's first token has been streamed to its caller, it
  admits the declared burst of long-prompt requests from one other named
  caller, so the pool the engine works from is claimed by one caller's later
  arrivals;
- when a named long request's prompt has been accepted by the engine, it holds
  every subsequent completion for that request at the transport layer until a
  second named request has been accepted, so the two occupy the pool together;
- when a named request has streamed half its declared answer length, it kills
  the service process and restarts it against the same engine;
- when a named request has been accepted, it restarts the engine itself, so
  everything the engine held is gone while the service still owes promises;
- it repeats a named request's identity after that identity's outcome has been
  reported.

The gate depends on a limit the environment does supply: the pool is fixed at
the size the lab config declares, and the controller does not have to invent
it. What the controller supplies is the transport-layer hold above, so the
regime is reached at the same records on every run.

## What must remain true

Every accepted identity has exactly one reported outcome, and an outcome
reported before a fault is the outcome after recovery. A request whose work
the engine repeated is not reported twice and does not receive two answers. A
request the service can no longer serve ends early with a stated reason
through the public interface, never by a dropped connection and never by a
silent truncation that reads as a finished answer.

Bound attainment is reported for each caller separately, so the burst appears
both as its own caller's cost and as what it cost the callers already
promised. The named request admitted before the burst keeps its promise or
ends early with a stated reason; a run in which no caller's attainment moves
relative to another's fails, because the pressure the lab is about was never
applied.

Answers match the independently produced ones by identity. The run reports the
observed pool occupancy and the work the engine repeated; a run in which the
pool never reaches the regime where the engine reclaims space fails rather
than passes, because the lab's subject was never met.

## What a weak design gets wrong

A weak design admits on queue length. The queue is in front of the engine and
the resource is inside it, so the number waiting says nothing about the space
remaining, and the design keeps accepting while the engine is already
reclaiming space from work it accepted earlier.

A weak design treats admission as the end of the decision. The promise is made
once, at accept time, from a reading that a later arrival invalidates — so the
service reports bounds it has already lost the ability to keep, and the first
the caller learns of it is silence.

A weak design counts an answer as finished because the stream stopped. Work
that was thrown away and work that completed look the same from the outside
unless the service distinguishes them itself.

A strong design decides admission against the resource that is actually
reclaimed, states a promise it can still defend after later arrivals, and
gives the caller an explicit ending when it cannot — and it says which of the
engine's properties it leaned on to know any of that.
