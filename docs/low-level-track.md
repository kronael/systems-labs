---
status: reference
---

# Low-level track

## Decision

Phase 6 is a second catalog in Rust and C. Its quirks live in the kernel and
the machine: I/O readiness and completion, durability, virtual memory, memory
allocation, and time. It is not a translation of phases 1 to 4.

A port is not a low-level lab. Rewriting the quote service in Rust inherits the
original's checks, failure schedule, and answer; the learner reimplements a
known solution under a stricter compiler and learns nothing new about the
machine. Every candidate below exposes a failure the high-level catalog cannot
reach.

The five candidates are unscored. They enter the catalog only after the core
catalog is `accepted`.

## Origination

Each candidate is grounded in a documented, publicly reported behavior rather
than an invented puzzle. The source column records where the quirk is
described. Every source here is *cited*: no prose, code, fixture, or test is
copied from it, and each lab's failure schedule and checks are original.

| # | Candidate | Quirk it falsifies | Origination |
|---|-----------|--------------------|-------------|
| 6/1 | Bounded-memory record shipper | A readiness or completion signal proves the data left your buffer | [io_uring(7)](https://man7.org/linux/man-pages/man7/io_uring.7.html) and [Efficient IO with io_uring](https://www.kernel.dk/io_uring.pdf), which fix that a CQE `res` carries a byte count like the syscall it replaces, and that a submitted buffer must stay valid until completion |
| 6/2 | Crash-safe record store | A failed `fsync` can be retried, and a successful `write` is durable | [fsyncgate 2018](https://danluu.com/fsyncgate/), the [pgsql-hackers thread](https://www.postgresql.org/message-id/CAMsr%2BYHh%2B5Oq4xziwwoEfhoTZgr07vdGG%2Bhu%3D1adXx59aTeaoQ%40mail.gmail.com), and [LWN's account](https://lwn.net/Articles/752063/). PostgreSQL's answer was `data_sync_retry`: do not retry, PANIC |
| 6/3 | Large index query service | `mmap` gives free, durable, well-scheduled file access | [Crotty, Leis, Pavlo, *Are You Sure You Want to Use MMAP in Your DBMS?*, CIDR 2022](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf): no control over write-back, single-threaded eviction, TLB shootdowns, and page-cache bandwidth below the device |
| 6/4 | Steady-state request service | Freeing memory returns it to the operating system | [`mallopt(3)`](https://man7.org/linux/man-pages/man3/mallopt.3.html) and [`malloc_trim(3)`](https://man7.org/linux/man-pages/man3/malloc_trim.3.html) fix that glibc releases heap memory only when contiguous free space at the top of the heap exceeds `M_TRIM_THRESHOLD`, that only allocations at or above the dynamically raised `M_MMAP_THRESHOLD` are independently returnable, and that thread heaps ignore the trim pad. [`malloc(3)`](https://man7.org/linux/man-pages/man3/malloc.3.html) adds arenas created on mutex contention, and the [transparent hugepage guide](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html) adds resident size inflated by 2 MB pages backing barely-touched regions |
| 6/5 | Rate-accurate replayer | A sleep-driven loop produces the rate you asked for | [Tene's 2013 mechanical-sympathy thread](https://groups.google.com/g/mechanical-sympathy/c/icNZJejUHfE) coins coordinated omission and reports a 99.99th percentile understated by four orders of magnitude; [wrk2](https://github.com/giltene/wrk2) measures each response from when its transmission should have occurred, and [HdrHistogram](https://github.com/HdrHistogram/HdrHistogram) implements the expected-interval correction. [`clock_nanosleep(2)`](https://man7.org/linux/man-pages/man2/clock_nanosleep.2.html) fixes that relative sleeps drift, that intervals round up to clock granularity, and that an absolute sleep on a settable clock returns early when the clock is stepped |

All five candidates now have full specs.

## Expanded candidates

### 6/1 Bounded-memory record shipper — specced

Ship an append-only record file to a slow remote consumer, in order, once per
acknowledged position, inside a fixed memory ceiling, resuming after restart.
The consumer stalls, reads a byte at a time, closes mid-record, and resets.

The learner discovers that neither an epoll readiness signal nor an io_uring
completion means the consumer has the data, that a submitted buffer is owned by
the kernel until completion, and that a partial transfer must be reconciled
against a record boundary. Memory is the forcing function: a growing input plus
a stalled consumer makes any unbounded buffer fail.

Spec: [`../6/1-bounded-memory-record-shipper.md`](../labs/6/1-bounded-memory-record-shipper/README.md).

### 6/2 Crash-safe record store — specced

Accept records, acknowledge only what survives power loss, serve them back in
order, and recover from a kill at an arbitrary instant.

The forcing case is the one `fsync` that returns `EIO` and then succeeds. On
Linux the failed write-back pages are already discarded and marked clean, so
the second call reports success over data that is gone. The store must treat
that error as terminal, exactly as PostgreSQL now does, and must also handle a
torn trailing record and a truncated file.

Spec: [`../6/2-crash-safe-record-store.md`](../labs/6/2-crash-safe-record-store/README.md).

### 6/3 Large index query service — specced

Serve point and range queries over an index far larger than resident memory,
under a declared tail-latency bound, while the file is being appended to.

The tempting design maps the file and treats access as memory. The paper's
findings then arrive in order: page-in stalls appear as request latency with no
I/O visible in the process's own accounting, eviction is single-threaded, TLB
shootdowns scale with cores, and the achieved read bandwidth stays far below
the device. `SIGBUS` on truncation and the absence of write-back control finish
the argument.

Spec: [`../6/3-large-index-query-service.md`](../labs/6/3-large-index-query-service/README.md).

### 6/4 Steady-state request service — specced

Serve requests for hours inside a fixed resident-memory ceiling with a declared
p99.9, where request sizes are heavy-tailed and lifetimes overlap.

Resident memory grows and does not come back down even though every allocation
is freed. The learner meets allocator arenas, fragmentation, trim thresholds,
and transparent huge pages, and finds that the tail latency is produced by
allocation rather than by the work the service performs.

Spec: [`../6/4-steady-state-request-service.md`](../labs/6/4-steady-state-request-service/README.md).

### 6/5 Rate-accurate replayer — specced

Replay a recorded event stream at a fixed offered rate against a system that
sometimes stalls, and report what the system did.

A sleep-per-event loop silently lowers the offered rate whenever the target
stalls, which is coordinated omission: the replayer stops measuring exactly
when the measurement matters. Timer granularity, scheduler preemption, and the
difference between `CLOCK_MONOTONIC` and wall time all move the result. This
candidate is the low-level counterpart to the open-loop generator that phases 1
to 5 supply as prepared infrastructure.

Spec: [`../6/5-rate-accurate-replayer.md`](../labs/6/5-rate-accurate-replayer/README.md).

## Scale contract

Each low-level lab carries a speed, a load, and an amount, per the
[lab brief contract](contract.md#lab-brief-contract). The numbers are
set when the lab is specced; they are chosen so the naive design fails on size
rather than on a reviewer's judgment.

## Governing references

- [`docs/contract.md`](contract.md) — course-wide learning,
  evidence, source, cost, and repository contracts.
- [`lab-selection.md`](lab-selection.md) — the scored selection that
  produced the original core ten.
- [`labs/README.md`](../labs/README.md) — authoritative list and lifecycle status.
