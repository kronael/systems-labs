# Bounded-memory record shipper

Design and build a log-shipping agent for one host: a service that reads the
records the host's applications append to a local log file and ships every
record to a remote collector over a stream socket, in order, exactly once per
accepted position. A stalled collector is the normal case, not the exception:
the collector is slower than the applications for long stretches and
sometimes stops reading entirely. The agent's memory is not its own — the
host exists to run the applications, so the agent's resident memory must stay
inside a fixed ceiling for the whole run. And the applications cannot be told
to slow down: a process appending a log line does not wait for the shipper
and has no channel to be asked to stop. After a restart the agent resumes
from its last acknowledged position.

The assignment is the whole agent: file and socket I/O, buffer ownership,
progress accounting, flow control, shutdown, and end-to-end tests. The I/O
model, the buffer-ownership strategy, the flow-control policy, and the
concurrency model are the learner's decisions.

## What you are given

The supplied Compose stack starts a collector harness whose read rate is
driven by a schedule file, a record generator that appends to the log file
the way the host's applications would — never waiting for the shipper —
telemetry collection, and the fault controller. The collector can stall, read
one byte at a time, close mid-record, and reset the connection at a named
record boundary.

The course supplies the wire framing, the record generator, the
acknowledgement protocol, and scenario files. A record is one opaque line;
there is nothing to parse. The learner owns the shipper and its tests. No
cloud account is required.

## Requirements

The service ships records in file order and never reports a position as
acknowledged before the collector has acknowledged it. A restart resumes from
the last acknowledged position without duplicating an already acknowledged
record and without skipping an unacknowledged one.

Resident memory has a declared ceiling that holds while the collector is
stalled for minutes. The ceiling is the host's claim, not a tuning choice:
the memory belongs to the workload the host is there to run. The service
continues to accept a growing log file during a stall; its memory use must not
grow with the size of that file, and it must never block or slow the
applications' writes.

Partial progress is normal: a write can accept fewer bytes than offered, and
the kernel can report progress for only part of what it was handed at once.
The service accounts for every byte and every record boundary across these
events. Configuration comes from the standard TOML contract.

The scale target is a 50 GB log file that keeps growing during the run, a
sustained 200 MB per second while the collector reads at full speed, a
collector stall of five minutes, and a resident-memory ceiling of 256 MB.
These numbers size the problem, not the pass bar: throughput is judged
against a relative baseline on the same host, and the memory ceiling is a
structural bound.

The evidence must separate bytes accepted by the kernel from bytes
acknowledged by the collector, and must show buffer occupancy over time. How
that measurement is produced is the learner's choice; no telemetry stack is
required.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what the kernel accepting bytes proves about their location and their
  delivery, and what it does not prove;
- who owns the memory holding a record from the moment it is handed to the
  kernel until it is safe to reuse, and what event proves that it is;
- how the memory ceiling is enforced when the collector stops reading;
- how a short or partial write is reconciled with a record boundary;
- what the resume position means, and why an earlier or later position is
  unsafe;
- how shutdown decides between draining, abandoning, and recording progress;
- which measurement separates a slow collector from a slow shipper.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

The collector receives every record once in file order across stalls,
mid-record closes, resets, and restarts. Resident memory stays inside the
declared ceiling during a multi-minute stall with a growing log file. The
applications' writes are never blocked or slowed by the shipper. Shutdown
records a position that the restart proves correct.

The evidence report includes throughput against collector read rate, peak and
steady resident memory, peak buffer occupancy, wake-up count per megabyte,
bytes accepted versus acknowledged over time, and the recovery position after
each injected failure. It ties the memory ceiling to a structural argument
rather than to an observed maximum.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **Fluent Bit** — [documentation](https://docs.fluentbit.io/manual/)
- **Vector** — [documentation](https://vector.dev/docs/)
- **Filebeat** — [documentation](https://www.elastic.co/docs/reference/beats/filebeat)

Stuck? See `hints/`.
