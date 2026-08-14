---
status: draft
---

# Crash-safe record store

## Brief

Design and build a local store that accepts records, returns success only when
the record will survive a power loss, and serves them back in insertion order.
The store must recover to a consistent state after the machine is killed at an
arbitrary instant, and it must state exactly which acknowledged records are
guaranteed to be present after that recovery.

The assignment is the whole store: on-disk layout, write path, acknowledgement
rule, recovery procedure, error policy, and end-to-end tests. The prompt does
not prescribe a file format, a journal design, a checksum scheme, a batching
rule, or a directory layout.

## Prepared scaffold

The supplied environment runs the store inside a container whose filesystem is
backed by a device that can be frozen at an exact write, drop unflushed data on
resume, return a single I/O error to one flush call and then behave normally,
and truncate a file between two operations. The fault controller triggers each
event at a named record boundary.

The course supplies the record generator, the client protocol, the crash
harness, scenario files, and the black-box grader. The learner owns the store
and its tests. No cloud account is required.

## Requirements

An acknowledged record survives a power-loss event. An unacknowledged record
may be present or absent after recovery, but the store must never come back
with a record that is partially written, silently truncated, or out of order,
and it must never lose a record that precedes a surviving later record.

The store detects corruption rather than serving it. A record whose content
does not match its recorded integrity value is reported as damaged; the store
does not return it as valid data.

A flush error is reported to the caller and is never retried into a false
success. The store defines what state it is in after a failed flush and whether
it continues to accept writes.

Recovery time has a declared bound expressed in terms of data volume. Startup
after an unclean stop must not require reading and validating unbounded
history. Configuration comes from the standard TOML contract.

The scale target is 100 GB of stored records, 50,000 acknowledged records per
second at steady state, and 64 concurrent writers. These numbers size the
problem, not the pass bar: acknowledgement rate is judged against a relative
baseline on the same host, and recovery after an unclean stop must complete
within the learner's declared bound at that stored volume.

The evidence must show flush latency, recovery duration against stored volume,
and records discarded during recovery. How that measurement is produced is the
learner's choice; no telemetry stack is required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- which operation makes a record durable, and what a successful write call
  proves before that operation;
- what must be durable in addition to the record's own bytes before an
  acknowledgement is honest;
- how a torn or partial trailing record is recognized and handled at startup;
- what happens to the store's guarantees when a flush returns an error once;
- how ordering is preserved across a crash without validating all history;
- how the recovery bound was derived, and what data volume it assumes;
- which durability claims the test environment can prove and which it cannot.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

The grader kills the process and the backing device at exact record boundaries,
drops unflushed data on resume, returns one I/O error to a flush call and then
resumes normal behavior, truncates the tail of a file, and flips bytes inside a
stored record. It then restarts the store and replays the full client history
against what the store serves.

The grader does not inspect private functions or require a named on-disk
format. It observes acknowledgements, served records, recovery duration, and
reported errors.

## Acceptance evidence

Every acknowledged record is present and in order after every injected crash.
No corrupted record is served as valid. The single flush error is surfaced to
the caller and is reflected in the store's stated state. Recovery stays inside
the declared bound.

The evidence report includes acknowledged-record survival per crash point,
write throughput against flush frequency, flush latency distribution, recovery
duration against stored volume, and the count of records discarded during
recovery. It names the durability property the container and device harness
cannot prove about real hardware.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **SQLite** in WAL mode makes commit durability a configuration choice: with
  `synchronous=NORMAL` a committed transaction may roll back after a power
  loss while the database stays consistent, and `synchronous=FULL` syncs the
  write-ahead log on every commit.
- **LMDB** copies pages on write and never overwrites active data, so it
  needs no special recovery procedure after a system crash; one write
  transaction is active at a time, commit flushes to disk by default, and the
  `MDB_NOSYNC` flag trades that away — a system crash can then corrupt the
  database or lose the last transactions.
- **RocksDB** journals every update to a write-ahead log, but by default a
  write returns once the data reaches the operating system: a process crash
  loses nothing, while a machine crash can lose the last updates unless the
  write asked for a sync.

Read their documentation on commit, sync, and recovery semantics. The lab
does not run them.

## Scope

The expected focused time is six to eight hours. The learner builds the store
and its tests. The crash harness, fault device, generator, protocol, and
telemetry are prepared. Replication, secondary indexes, compaction across
generations, transactions over multiple records, and networked clients are
outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/2-low-level-track.md`](../0/2-low-level-track.md) — low-level track
  rationale and candidates.
- [fsyncgate](https://danluu.com/fsyncgate/) — the PostgreSQL thread showing
  that a failed `fsync` can clear the error and the dirty page, so the next
  `fsync` reports success over data that never reached the disk.
- [PostgreSQL error handling in fsync](https://lwn.net/Articles/752063/) — the
  kernel-side account of where the error is reported and where it is lost.
- Implementation pointers do not exist while the spec is `draft`.
