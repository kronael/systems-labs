> Spoilers. Open only when stuck.

# Rate-accurate replayer — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Verification first runs the replayer against an instantly answering target to
calibrate its schedule fidelity. The failure schedule then replays the same
configuration under faults: the target stalls for ten seconds beginning when a
named event is acknowledged, raises its service time a hundredfold across a
named span of events, and refuses new connections at a named boundary; the
harness steps the container's wall clock at a named event; and SIGTERM
arrives once a named event has been sent, while others are in flight.

Checks do not inspect private functions or require a named timing interface,
concurrency model, or distribution format. They observe the time every event
actually left the replayer, the harness's own receive and acknowledgement log,
resident memory, and the report.

Verification computes the distribution a client arriving on the declared
schedule would have observed from observation, never from the service script.
Each event's due time under the declared schedule, the instant it left the
replayer, and the instant its response or its refusal arrived are observed
independently of the replayer's own report, on one monotonic time base whose
commonality across the replayer, the harness, and the controller the run
establishes before the scenario and re-checks after it. An event's latency is
its terminal instant minus its due time, and an event still unanswered at its
due-time deadline is timed out, the deadline winning the tie. The scripted
service times describe the target's behaviour and enter no expected latency:
the host adds delay no script models, because a thread whose sleep has
completed still waits for a CPU, and a requested interval rounds up to the
granularity of the underlying clock.

The clock layer is the layer this lab's required gate depends on: due times
are computed on a time source the run controls, and every observation the gate
compares is stamped on the one base the run proved common. The ten-second
stall the gate measures is a transport-layer hold on the target's acceptance,
released at its declared boundary. A run whose time bases are not shown
common, or in which the stall never activates, fails rather than passes.
