# Dataset release service

Design and build a service that publishes a large dataset as a series of
releases. Publishers submit a full release, and between full releases they
submit numbered increments. Consumers ask the service what is current, then
fetch that release or any increment by its identity and get the exact bytes.

The assignment is the whole service: the publishing path, the public read of
what is current, the fetch path, the accounting for work that is under way,
the telemetry, and end-to-end tests. How a release is laid out in the store,
how a release becomes current, how an interrupted publication resumes, and how
the service divides into processes are the learner's decisions.

## What you are given

The supplied Compose stack starts an S3-compatible object store, the seeded
release and increment generator, OpenTelemetry collection, a metrics backend,
and the fault controller. The generator produces reproducible payload streams
with checksums, so a check can compare bytes by identity without storing a
second copy.

The store is the environment, and its limits are environment facts. A single
write request carries at most 5 GB, and a server-side copy is atomic only up
to 5 GB, so a release above that size arrives in parts. The controller stands
between the service and the store: it refuses what those limits forbid with
the store's own error shape, and it admits at most 64 MiB per second, which
stands in for the network a local store does not have.

The learner owns the service and its Compose layer. The common Makefile starts
dependencies, validates scaffold health, runs tests, injects faults, runs load,
and collects evidence. No cloud account is required, and this lab has no
`make smoke`: the hosted comparison is not part of the course's optional cloud
surface yet, so the gate proves a declared model with nothing hosted to check
it against, and `EVALUATION.md` lists what that leaves unverified.

## Requirements

The service answers what is current as one value: a release identity and the
increment number that goes with it. Four things must hold at that public read.

Current never moves backwards, for any reader, at any moment. Everything
current names is fetchable immediately and byte-exact. A release that is no
longer current stays fetchable by its own identity for the whole run. And at
the end of a run the store holds no bytes belonging to work the service will
never finish; whatever is still under way is reported rather than abandoned.

A consumer that reads current and then fetches what it named must receive
those bytes. It may never receive a partial answer presented as a whole one,
and it may never be told that what current named is absent.

Increments are numbered and ordered. A consumer that has an increment number
can ask for the next one and either receive it whole or be told it does not
exist yet.

The scale target is releases of 12 GiB with three published per run, one
increment of about 1 MiB every sixty seconds with at least sixty per run, and
eight consumers reading current every five seconds throughout. The numbers are
the shape of public dataset publishing: the weekly OpenStreetMap planet file
is about 88 GB and its increments are published every minute, so the release
is far larger than one request can carry while the increments are small and
continuous. Throughput is judged against the admitted budget and the learner's
declared bounds, not a fixed number.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what makes a release current, and what a reader can observe while that is
  happening;
- what the service knows, after it restarts, about a publication that was
  under way when it stopped;
- where the knowledge needed to resume a publication lives, and why it
  survives the process;
- how two publishers that both believe they are publishing are kept from
  producing a current value that regresses;
- what the service can say about bytes it has sent to the store but has not
  published, and how it accounts for them;
- which properties of the store the design relies on, and which it explicitly
  does not.

At least two designs must be compared for how a release becomes current. The
chosen design needs stated failure modes and one residual limitation.

## Acceptance evidence

Every read of current across the whole run is non-decreasing, checked by
identity and by increment number. Every fetch of what current named returned
whole, byte-exact content. Every superseded release remained fetchable by
identity to the end of the run.

The evidence report includes the bytes admitted to the store per release, the
bytes belonging to unfinished work at the end of the run, the time from a
publisher's first byte to the release becoming current, and the consumers'
observed sequence of current values. It explains the relationship between the
admitted budget and the publication time, and ties the design's resumption
decision to the measurements.

A design that republishes a release from the beginning after an interruption
must show what that cost, and the evidence must make the cost visible rather
than average it away.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does not
run them. What each does differently at this lab's boundary is in `hints/`,
because saying it here would point straight at the answer.

- **Apache Iceberg** — [documentation](https://iceberg.apache.org/docs/latest/)
- **lakeFS** — [documentation](https://docs.lakefs.io/)
- **DVC** — [documentation](https://dvc.org/doc)

## What is outside the problem

The expected focused time is eight to twelve hours. The learner builds the
service and its tests. The store, the generator, the admitted budget,
telemetry, and fault injection are prepared. Dataset formats, compression,
diff computation, access control, billing, and any user interface are outside
the problem. Three completed releases occupy 36 GiB of local disk, and a
design that leaves unfinished work behind occupies more.

Stuck? See `hints/`.
