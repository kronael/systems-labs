# Steady-state request service

Design and build a service that gives each client a private in-memory
workspace: a client opens a workspace, stores named entries in it, reads them
back byte for byte, and closes it, discarding everything it held. Workspaces
overlap: most live for seconds, some stay open for hours. Entry sizes are
heavy-tailed, from tens of bytes to megabytes. The service must hold a declared
tail-latency bound and a declared resident-memory ceiling across a run measured
in hours, through a peak that multiplies live data and then subsides.

The assignment is the whole service: connection handling, workspace state,
request path, memory accounting, shutdown, and end-to-end tests. The memory
strategy, the data layout, the concurrency model, and the accounting scheme
are the learner's decisions.

## What you are given

The supplied Compose stack starts the workload generator, telemetry
collection, and the fault controller. The generator drives the request
schedule open-loop from a seeded profile: it opens and closes workspaces on
declared lifetime distributions, draws entry sizes from a declared
heavy-tailed distribution, and changes phase at named workspace identities.
The service container runs under a memory limit set to the declared ceiling,
so exceeding it ends the run.

The course supplies the request protocol, the generator, and scenario files.
The learner owns the service and its tests. No cloud account is required.

## Requirements

A read returns exactly the bytes most recently stored under that workspace and
entry identity, and never bytes from any other entry, live or discarded. After
a workspace closes, its entries are gone: a later read fails with the stated
error. One process start serves the entire run; exiting to shed memory is a
failure, not a recovery.

Resident memory must track the work rather than remember it. After the peak
phase ends, resident memory falls back inside a declared bound stated in terms
of live data, and from then on it does not drift upward while load and live
data stay level. The p99.9 response time has a learner-declared bound that
holds in every phase, including while the peak is being built and while it is
being given back. Configuration comes from the standard TOML contract.

The scale target is a sustained 20,000 requests per second for a two-hour run,
512 concurrent client connections with 100,000 workspaces open at the peak,
and live entry data that peaks at 4 GB and then falls to 512 MB for the
remainder. These numbers size the problem; the pass thresholds stay relative,
structural, or learner-declared, per the course contract.

The evidence must show resident memory and live entry data as two aligned
curves over the whole run, and the response-time distribution per phase. How
that measurement is produced is the learner's choice; no telemetry stack is
required.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what releasing memory actually returns it to, and under what condition any
  of it reaches the operating system;
- what the resident number counts, how it differs from live data, and which
  of the two the ceiling constrains;
- how the sizes and lifetimes in this workload decide whether released space
  is reusable for the next request, returnable to the system, or neither;
- how serving many connections at once changes where memory is released from,
  and what that does to how much the process retains;
- which requests pay the p99.9, and why the price is set by obtaining memory
  rather than by serving entries;
- how the ceiling is enforced by construction rather than observed after the
  fact;
- which platform mechanisms can raise resident memory without a single new
  allocation, and whether this environment enables them.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

Every read in the full history returns the exact stored bytes for its
identity; no read observes another entry's bytes or a closed workspace's data.
Resident memory returns to the declared bound after the peak and shows no
upward drift across the steady hours. The p99.9 stays inside the declared
bound in every phase. Shutdown drains cleanly.

The evidence report includes the resident-versus-live curve pair, the ratio
between the two at peak and at steady state, achieved throughput against
offered rate, and the response-time distribution per phase with the tail
attributed to the requests and phase that paid it. It ties the declared
ceiling to a structural argument rather than an observed maximum, names the
platform mechanism that most limits how far resident memory can fall, and
states the residual limitation.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **jemalloc** — [documentation](https://jemalloc.net/)
- **tcmalloc** — [documentation](https://google.github.io/tcmalloc/)
- **mimalloc** — [documentation](https://github.com/microsoft/mimalloc)

Stuck? See `hints/`.
