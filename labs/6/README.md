---
status: reference
---

# Phase 6 — Low-level

This page orients the phase: what its labs study, and why they sit together.
The labs themselves are the directories beside it.

## What this phase is about

Phase 6 is a separate catalog in Rust and C whose quirks live in the kernel
rather than in a database or broker: I/O readiness and completion,
durability, virtual memory, memory allocation, and time. A port of a phase
1–5 lab is not a low-level lab, because it inherits the original's
checks, failure schedule, and answer; every lab here exposes a failure
the other tracks cannot reach, and each is grounded in a documented, publicly
reported behavior recorded in the track record `CONTRIBUTING.md` points to.

## The labs

- [Bounded-memory record shipper](1-bounded-memory-record-shipper/README.md) — a
  host log-shipping agent that forwards a growing file to a slow, stalling
  collector, in order, inside a fixed memory ceiling it does not own.
- [Crash-safe record store](2-crash-safe-record-store/README.md) — a local store
  that acknowledges only what survives power loss and recovers from a kill
  at an arbitrary instant.
- [Large index query service](3-large-index-query-service/README.md) — a query
  service over a record set far larger than resident memory, defending a
  declared tail-latency bound.
- [Steady-state request service](4-steady-state-request-service/README.md) — a
  workspace service that holds a resident-memory ceiling and a p99.9 across
  an hours-long run with a peak that subsides.
- [Rate-accurate replayer](5-rate-accurate-replayer/README.md) — a replayer that
  drives a recorded stream on a declared schedule and reports what the
  target did, trustworthy to the far tail.

## The technologies

**Rust** is one of the two learner languages. The phase chooses it because
its ownership model makes the questions these labs ask — who owns a buffer,
when memory becomes reusable, what a lifetime spans — explicit in the type
system rather than implicit in discipline. Documentation:
[doc.rust-lang.org](https://doc.rust-lang.org/).

**C** is the other learner language, with glibc as its runtime library. The
phase chooses it because nothing stands between the program and the
interfaces under study: the allocator, the page cache, and the syscall
boundary are the program's direct environment. Documentation:
[GNU C Library manual](https://sourceware.org/glibc/manual/).

**The Linux kernel** is the laboratory, and the
[Linux man-pages project](https://www.kernel.org/doc/man-pages/) is this
phase's primary reading: it documents the kernel and C library interfaces
whose exact contracts the labs turn into requirements. The interfaces the
specs name:

- **epoll and io_uring** — the readiness and completion models for
  asynchronous I/O. The record shipper lives on what these signals do and do
  not prove.
  [epoll(7)](https://man7.org/linux/man-pages/man7/epoll.7.html),
  [io_uring(7)](https://man7.org/linux/man-pages/man7/io_uring.7.html).
- **fsync and the page cache** — the boundary between a successful write and
  durable data, which the crash-safe store must state honestly.
  [fsync(2)](https://man7.org/linux/man-pages/man2/fsync.2.html).
- **mmap and madvise** — file access through virtual memory, the tempting
  design the large-index lab measures against its alternatives.
  [mmap(2)](https://man7.org/linux/man-pages/man2/mmap.2.html),
  [madvise(2)](https://man7.org/linux/man-pages/man2/madvise.2.html).
- **The glibc allocator** — arenas, thresholds, and the gap between freed
  and returned memory that the steady-state service must close, with
  transparent huge pages able to raise residency without a new allocation.
  [malloc(3)](https://man7.org/linux/man-pages/man3/malloc.3.html),
  [mallopt(3)](https://man7.org/linux/man-pages/man3/mallopt.3.html),
  [transparent hugepage support](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html).
- **Clocks and sleeping** — timer granularity, drift, and clock bases,
  which decide whether a replayer's schedule survives a stalling target.
  [clock_nanosleep(2)](https://man7.org/linux/man-pages/man2/clock_nanosleep.2.html).

**Docker Compose** still carries the harnesses: consumer and target
harnesses, generators, telemetry, the fault controller, and the containers
whose memory limits and fault-injected block devices make the kernel's
behavior reproducible. Documentation:
[docs.docker.com/compose](https://docs.docker.com/compose/).

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names what a practitioner would
have reached for instead — Fluent Bit, Vector, Filebeat,
SQLite, LMDB, RocksDB, ScyllaDB, jemalloc, tcmalloc, mimalloc, wrk,
tcpreplay, k6 — and the one thing each does differently at that lab's
boundary. They appear as reading rather than as dependencies: the labs never
run them, and knowing what each trades away is part of defending a design.

## Cloud

This phase needs no cloud account and touches no cloud service: the local
Linux kernel is the laboratory, and every gate runs on it. The curriculum's
optional cloud path — Lambda, SQS, DynamoDB on-demand, and short-retention
logs, with MSK, EKS, NAT gateways, RDS, and ElastiCache excluded by policy
because they bill by the hour — belongs to phase 4, not here. See
[cloud access](../cloud-access.md).
