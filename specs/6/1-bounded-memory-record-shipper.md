---
status: draft
---

# Bounded-memory record shipper

## Brief

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

## Prepared scaffold

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
continues to accept a growing log file during a stall; it must not read the
entire file into memory, and it must never block or slow the applications'
writes.

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

## Architecture questions

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

## Adversarial evaluation

The failure schedule stalls the collector for a fixed interval at an exact
record boundary, resumes it at a byte-per-read rate, closes the connection
mid-record, resets the connection after acknowledging a known position, and
sends SIGTERM while a submission is outstanding. It grows the log file
during every stall.

Checks do not inspect private functions or require a named I/O interface.
They observe the received byte stream, the acknowledgement sequence, the
resume position after restart, resident memory over time, and telemetry.

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

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **Fluent Bit** — [documentation](https://docs.fluentbit.io/manual/). Caps
  buffered data with a per-input memory limit and pauses ingestion when the
  limit is reached; its backpressure page warns that some inputs "are prone
  to data loss" once that happens under memory-only buffering, and offers
  the filesystem as the buffer that survives.
- **Vector** — [documentation](https://vector.dev/docs/). Makes the
  full-buffer decision an operator setting on each destination: block until
  there is room, pushing the stall upstream, or drop the newest events —
  with a disk buffer as the variant that survives a restart.
- **Filebeat** — [documentation](https://www.elastic.co/docs/reference/beats/filebeat).
  Treats the log file itself as the buffer: when the output stalls it stops
  reading, keeps its position in a registry file, resumes when the output
  recovers, and delivers at least once — a shutdown mid-send re-delivers
  after restart.

The lab does not run them.

## Scope

The expected focused time is twelve to sixteen hours. The learner builds the
shipper and its tests. The collector harness, framing, generator, telemetry,
and fault schedules are prepared. Log formats, parsing, multiline handling,
file rotation, encryption, compression, multiple collectors, and distributed
coordination are outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/2-low-level-track.md`](../0/2-low-level-track.md) — low-level track
  rationale and candidates.
- [Fluent Bit: Backpressure](https://docs.fluentbit.io/manual/administration/backpressure)
  — the real reported behaviour behind the brief: an agent limits how much a
  source "can buffer to memory", pauses the input when that limit is
  reached, and "some input plugins are prone to data loss after
  `mem_buf_limit` capacity is reached during memory-only buffering".
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [`epoll`](https://man7.org/linux/man-pages/man7/epoll.7.html) — the
  readiness half of the readiness-versus-completion distinction: an event
  says the "file descriptor is ready for the requested I/O operation", the
  application still performs the transfer itself, and the buffer never
  leaves its hands. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- [`io_uring`](https://man7.org/linux/man-pages/man7/io_uring.7.html) — a
  submitted operation owns its buffer until its completion is reaped, so
  in-flight work, not the application's queue, sets the memory floor.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [Efficient IO with io_uring](https://kernel.dk/io_uring.pdf) — the ring
  design and what submission and completion mean for ownership and ordering.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
