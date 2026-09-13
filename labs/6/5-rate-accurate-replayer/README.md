# Rate-accurate replayer

Design and build a replayer that drives a recorded event stream into a target
service on a declared schedule and reports what the target did to every event.
The schedule is derived from the recording — original spacing, scaled spacing,
or a fixed rate — and every event's due time is computable before the run
begins. The target sometimes answers in microseconds, sometimes takes seconds,
and sometimes stops answering entirely; the declared schedule does not change
when it does.

The report must describe the experience of clients arriving on that schedule,
not the experience the replayer found convenient to measure. Every event has
exactly one reported outcome, and the latency distribution must stay
trustworthy at its far tail, because the far tail is the reason the run exists.

The assignment is the whole tool: pacing, transport, in-flight accounting,
timeout policy, measurement, the report, shutdown, and end-to-end tests. The
pacing strategy, the concurrency model, the connection strategy, and the
measurement design are the learner's decisions.

## What you are given

The supplied Compose stack starts a target harness whose per-event service time
follows a seeded script describing the target's own behaviour, a recording
generator, telemetry collection, and the fault controller. The harness
acknowledges each delivered event by identity, and it can stall for a fixed
interval starting at a named event, raise its service time across a named
span, and refuse new connections at a named boundary.

The endpoint the stack publishes is served through the fault controller's
transport layer, which is where a declared stall holds the target's acceptance
and releases it at the declared boundary while the replayer keeps running. The
harness's receive and acknowledgement log carries the same monotonic time base
the stack publishes to every process in the run.

The course supplies the wire protocol, the recordings, and scenario files.
The learner owns the replayer and its tests. No cloud account is required.

## Requirements

Every event in the recording is offered exactly once, at its due time under the
declared schedule. Due times are fixed before the run starts and do not move
when the target slows, stalls, or refuses connections: the offered schedule
during a fault must match the offered schedule the same configuration produces
against an instantly answering target.

The report accounts for every event by identity with exactly one outcome:
acknowledged, timed out, or refused. The run configuration declares a per-event
timeout, and an event's deadline is its due time plus that timeout. The
reported latency of an event is the time a client arriving at that event's due
time would have waited: the instant that event's outcome became terminal,
minus its due time, both read from the run's single monotonic time base. An
event still unanswered at its deadline is timed out, and the deadline wins the
tie. The distribution over those values is what the report summarizes.
Latencies in one run span microseconds to tens of seconds; the report states
percentiles to p99.99 and the maximum, with a declared value precision that
holds across that whole span.

The replayer separates its own contribution from the target's: the deviation of
each send from its due time is measured and reported as its own distribution.
An adjustment of the host's wall clock during a run must not move a due time or
corrupt a recorded latency. The recording is read as a stream, resident memory
has a declared ceiling that holds while the target stalls at the full offered
rate, and configuration comes from the standard TOML contract.

The scale target is a recording of 500 million events, a sustained offered rate
of 100,000 events per second, and a ten-second target stall at that rate — one
million events falling due with nothing accepting them — inside a
resident-memory ceiling of 512 MB. These numbers size the problem so a design
that only works when the target cooperates fails on its own terms. They are not
pass thresholds: thresholds stay relative to the calibration run, inside the
declared precision, or learner-declared.

This lab's evidence is the measurement itself, so the report must contain the
achieved offered rate over time, the send-deviation distribution, the latency
distribution per scenario, and the outcome of every event. How those numbers
are produced is the learner's choice; that the report contains them is not.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what the declared rate promises when the target stops answering;
- whose delay the reported latency of each event contains, and how the
  replayer's own contribution is separated and shown;
- what limits how precisely a process can wait for a short interval, and how
  the schedule is met at 100,000 events per second despite that limit;
- which time base defines the schedule, and why a wall-clock adjustment must
  not move a due time;
- how in-flight state stays inside the memory ceiling through a ten-second
  stall, and what outcome an event that falls due during the stall receives;
- why the far tail of the reported distribution deserves trust: how the value
  precision was chosen, and which check would expose a misleading percentile;
- how a shutdown mid-run keeps the report honest about events still in flight.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

Every event in the recording appears exactly once, by identity, across the
acknowledgement log, the timed-out list, and the refused list. The send
schedule under every fault matches the calibration run within the tolerance
calibration established, including through the stall and the clock step. The
reported percentiles match the distribution independently computed from the
observed due, send, and arrival times, within the declared value precision at
every reported quantile, including p99.99 and the maximum of the stall run.
Resident memory stays inside the ceiling through the stall. The report
produced after SIGTERM accounts for every event that had fallen due.

The evidence report includes the achieved offered rate over time, the
send-deviation distribution, the latency distribution per scenario at the
declared precision, the outcome of every event, peak in-flight count, and
resident memory over time. It states the schedule tolerance and the value
precision as declared bounds and shows both held, records the check that
established one common time base across the run, and names the largest
disagreement between the report and the independently observed truth and
explains where it comes from.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **wrk** — [documentation](https://github.com/wg/wrk)
- **tcpreplay** — [documentation](https://tcpreplay.appneta.com/)
- **k6** — [documentation](https://grafana.com/docs/k6/latest/)

## What is outside the problem

The expected focused time is fourteen to eighteen hours. The learner builds
the replayer and its tests. The target harness, wire protocol, recordings,
and scenario files are prepared. Multi-host load generation, protocol realism
beyond the supplied framing, TLS, retries, response-body validation, and live
capture of new recordings are outside the problem.

Stuck? See `hints/`.
