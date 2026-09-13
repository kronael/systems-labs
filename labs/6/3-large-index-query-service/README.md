# Large index query service

Design and build a service that answers point and range queries over a keyed
record set far larger than resident memory, stored in segment files that the
prepared environment appends to, seals, and retires throughout the run. Every
answer is exact, the service declares and defends a tail-latency service
level, and resident memory stays inside a fixed ceiling while queries run on
every available core.

The assignment is the whole service: file access, memory policy, I/O
scheduling, concurrency, latency accounting, and end-to-end tests. The
file-access strategy, the memory policy, and the concurrency model are the
learner's decisions.

## What you are given

The supplied environment runs the service in a container with a fixed memory
limit, beside a data harness that generates the seeded segment files, appends
records to the active segment during the run, seals segments at named record
boundaries, and truncates retired segments at named barriers. A manifest
published by the harness states which records are live, and the query
protocol defines the outcome a retired record must receive.

The course supplies the segment format, the query protocol, the open-loop
load generator, a tool that records the host device's read capacity as a
baseline, and scenario files. The learner owns the service and its tests. No
cloud account is required.

## Requirements

A point query returns the record with the requested key, or a stated absence.
A range query returns exactly the live records in the requested interval, in
key order. A query that touches a retired segment receives the protocol's
retirement outcome rather than a crash, a stale answer, or a partial record.
No answer ever contains a corrupt record.

Records appended during the run become queryable within a declared visibility
bound, and the service states that bound rather than hiding it.

Resident memory has a declared ceiling far below the data volume, and the
ceiling holds for the whole run, including while every core serves queries.
Range scans and point queries run concurrently; a scan must not starve the
point-query tail. Configuration comes from the standard TOML contract.

The scale target is 100 GB of indexed records growing throughout the run, a
resident-memory ceiling of 4 GB, and a sustained 50,000 point queries per
second from 64 concurrent streams while range scans walk the older segments.
These numbers size the problem; they are not pass thresholds. Thresholds stay
relative to the recorded device baseline, structural, or learner-declared.

The evidence must show throughput and latency across the moment the touched
data first exceeds resident memory, must separate time waiting for data from
time executing, and must compare achieved read bandwidth with the recorded
device baseline. This lab requires that measurement: the tail is the
evidence, and a design that cannot see its own waiting cannot defend its
service level.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what a read of non-resident data costs, and in which component's accounting
  that cost appears;
- which component decides what stays in memory under the ceiling, and what
  evidence shows the decision is being followed under pressure;
- how the design knows whether answering a query will wait on the device
  before it commits a thread to that wait;
- which costs of the chosen access path grow with the number of cores rather
  than with the amount of data, and how the evidence isolates them;
- how a file that shrinks under a live read is prevented from crashing the
  service or corrupting an answer;
- how appended records become visible, and what bounds the staleness;
- why the achieved read bandwidth differs from the device baseline, and what
  would close the gap.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

Every answer matches the seeded truth, including during appends, at the
retirement barrier, and after restart. The queries in flight at the
truncation barrier receive the retirement outcome, by exact query identity,
and the service keeps answering. Resident memory never leaves the declared
ceiling. After the touched data exceeds memory, sustained load shows no
interval in which the service stops answering.

The evidence report includes throughput and per-class latency across the
memory transition against the declared service level, resident memory against
the ceiling, achieved read bandwidth against the device baseline at matching
concurrency, the separation of waiting from executing in the tail, and the
cost that grew when the stream count reached every core. It names the query
class that would miss its service level first under further growth.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`HINTS.md`, because saying it here would point straight at the answer.

- **LMDB** — [documentation](http://www.lmdb.tech/doc/)
- **RocksDB** — [documentation](https://rocksdb.org/docs/getting-started.html)
- **ScyllaDB** — [documentation](https://docs.scylladb.com/)

Stuck? See `HINTS.md`.
