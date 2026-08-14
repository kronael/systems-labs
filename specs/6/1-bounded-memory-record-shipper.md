---
status: draft
---

# Bounded-memory record shipper

## Brief

Design and build a service that reads an append-only record file and ships
every record to a remote consumer over a stream socket, in order, exactly once
per accepted position. The consumer is slower than the producer for long
stretches and sometimes stops reading entirely. Process memory must stay inside
a fixed ceiling for the whole run, and the service must resume from its last
acknowledged position after a restart.

The assignment is the whole service: readiness handling, buffer ownership,
progress accounting, flow control, shutdown, and end-to-end tests. The prompt
does not prescribe an event loop, an I/O interface, a buffer strategy, a thread
model, or a batching rule.

## Prepared scaffold

The supplied Compose stack starts a consumer harness whose read rate is driven
by a schedule file, a record generator, telemetry collection, and the fault
controller. The consumer can stall, read one byte at a time, close mid-record,
and reset the connection at a named record boundary.

The course supplies the wire framing, the record generator, the acknowledgement
protocol, scenario files, and the black-box grader. The learner owns the
shipper and its tests. No cloud account is required.

## Requirements

The service ships records in file order and never reports a position as
acknowledged before the consumer has acknowledged it. A restart resumes from
the last acknowledged position without duplicating an already acknowledged
record and without skipping an unacknowledged one.

Resident memory has a declared ceiling that holds while the consumer is
stalled for minutes. The service continues to accept a growing input file
during a stall; it must not read the entire file into memory, and it must not
block the producer's writes.

Partial progress is normal: a write can accept fewer bytes than offered, and a
completion notification can arrive for a subset of a submitted batch. The
service accounts for every byte and every record boundary across these events.
Configuration comes from the standard TOML contract.

The scale target is a 50 GB input file that keeps growing during the run, a
sustained 200 MB per second while the consumer reads at full speed, a consumer
stall of five minutes, and a resident-memory ceiling of 256 MB. These numbers
size the problem, not the pass bar: throughput is judged against a relative
baseline on the same host, and the memory ceiling is a structural bound.

The evidence must separate bytes accepted by the kernel from bytes acknowledged
by the consumer, and must show buffer occupancy over time. How that measurement
is produced is the learner's choice; no telemetry stack is required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what a readiness or completion signal proves about the data's location, and
  what it does not prove;
- who owns each buffer between submission and completion, and when that memory
  becomes reusable;
- how the memory ceiling is enforced when the consumer stops reading;
- how a short or partial write is reconciled with a record boundary;
- what the resume position means, and why an earlier or later position is
  unsafe;
- how shutdown decides between draining, abandoning, and recording progress;
- which measurement separates a slow consumer from a slow shipper.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

The grader stalls the consumer for a fixed interval at an exact record
boundary, resumes it at a byte-per-read rate, closes the connection mid-record,
resets the connection after acknowledging a known position, and sends SIGTERM
while a submission is outstanding. It grows the input file during every stall.

The grader does not inspect private functions or require a named I/O
interface. It observes the received byte stream, the acknowledgement sequence,
the resume position after restart, resident memory over time, and telemetry.

## Acceptance evidence

The consumer receives every record once in file order across stalls, mid-record
closes, resets, and restarts. Resident memory stays inside the declared ceiling
during a multi-minute stall with a growing input. The producer is never blocked
by the shipper. Shutdown records a position that the restart proves correct.

The evidence report includes throughput against consumer read rate, peak and
steady resident memory, peak buffer occupancy, wake-up count per megabyte,
bytes accepted versus acknowledged over time, and the recovery position after
each injected failure. It ties the memory ceiling to a structural argument
rather than to an observed maximum.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **epoll** signals readiness rather than completion: an event says the
  descriptor is ready for the requested I/O, the application still performs
  the write itself, and the buffer never leaves the application's hands —
  with edge-triggered use requiring writes until `EAGAIN` before waiting
  again.
- **POSIX AIO** offers a completion-shaped interface, but the Linux
  implementation lives in user space in glibc, simulated with threads that
  are expensive to maintain and scale poorly, and a submitted buffer must not
  be changed while the operation is in progress.
- **A blocking thread per connection** turns back-pressure into a blocked
  thread: `send` normally blocks when the message does not fit into the
  socket's send buffer, and a successful return carries no indication of
  delivery to the peer.

Read their documentation on readiness, completion, and buffer ownership. The
lab does not run them.

## Scope

The expected focused time is six to eight hours. The learner builds the shipper
and its tests. The consumer harness, framing, generator, telemetry, and fault
schedules are prepared. Encryption, compression, multiple consumers, and
distributed coordination are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/2-low-level-track.md`](../0/2-low-level-track.md) — low-level track
  rationale and candidates.
- [`io_uring`](https://man7.org/linux/man-pages/man7/io_uring.7.html) — a
  submitted operation owns its buffer until its completion is reaped, so
  in-flight work, not the application's queue, sets the memory floor.
- [Efficient IO with io_uring](https://kernel.dk/io_uring.pdf) — the ring
  design and what submission and completion mean for ownership and ordering.
- Implementation pointers do not exist while the spec is `draft`.
