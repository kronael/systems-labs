# TODO

Author-facing. `BUGS.md` holds what is wrong with what exists; this file holds
what does not exist yet. A proposal moves out of here by becoming a spec under
`specs/<phase>/`, and it may not become one until its quirk is grounded — see
`CONTRIBUTING.md`, "Adding a lab", step 1.

## The scaffold

Nothing is built. `specs/0/5-shared-scaffold.md` specifies the generator, the
fault controller, the evidence writer and `template/`, and the approval
boundary blocks all of it while `specs/01-systems-labs.md` is `draft`. The
controller's three layers — transport, process, and clock — are specified now,
and every lab that depends on one names it, but none of it is built. That work
comes before any new lab.

## Gaps in the catalog

Each entry below names a failure no existing lab reaches. The first three
carry a documented, publicly reported behaviour, fetched and quoted. The last
two do not, and must not be authored until they do.

### Time that goes backwards — grounded

No lab makes a system depend on comparing two clock readings, and none has a
fault that moves the clock. Thirty-four labs measure latency, rank backends by
observed speed, and expire leases, all on the belief that the difference
between two readings cannot be negative.

Cloudflare's 2017 outage is that belief failing in production: "The root cause
of the bug that affected our DNS service was the belief that *time cannot go
backwards*." A leap second made a round-trip time negative, and "`rand.Int63n`
promptly panics if its argument is negative"
([Cloudflare](https://blog.cloudflare.com/how-and-why-the-leap-second-affected-cloudflare-dns/),
read 2026-09-12).

The product: a service that dispatches each request to whichever of several
interchangeable backends has been answering fastest, and keeps serving while
the measurement itself becomes untrustworthy. The fault fires at a named
request, not a timer: the clock steps backwards between two readings the
design compares. Phase 1, because the learner operates the clock source.

### Two versions of the same service at once — grounded

No lab runs the learner's own code at two versions simultaneously. Every
recovery in the catalog restarts the same binary. A rolling upgrade does not:
old and new serve the same traffic against the same store while a contract
changes underneath them.

Kubernetes documents the constraint as a supported operating state rather than
an accident: `kubelet` "may be up to three minor versions older than
`kube-apiserver`", and the controllers "must not be newer than the
`kube-apiserver` instances they communicate with", "to allow live upgrades"
([Kubernetes](https://kubernetes.io/releases/version-skew-policy/), read
2026-09-12).

The product: a service that changes the shape of what it stores while it keeps
serving, with both versions live and neither able to assume it wrote what it
reads. The fault fires when a named record is written by one version and read
by the other. Phase 4, beside `4/5`, which already owns the deployment surface.

### Accepted work discarded to make room — grounded

No lab has a dependency that throws away work it already accepted and
committed compute to. The closest, `6/1`, bounds memory by refusing new work;
this refuses nothing and destroys what is in flight.

vLLM documents it as normal operation: a request is "preempted by
PreemptionMode.RECOMPUTE mode because there is not enough KV cache space", and
preempted requests "get recomputed when sufficient KV cache space becomes
available again"
([vLLM](https://docs.vllm.ai/en/latest/configuration/optimization.html), read
2026-09-12). Cached blocks are evicted least-recently-used from a fixed pool
([vLLM](https://docs.vllm.ai/en/latest/design/v1/prefix_caching.html), read
2026-09-12).

The product: a streaming generation API that promises a first-token bound and
a completion bound to every accepted request, over a fixed memory pool where
one long request can cause another's finished work to be thrown away. The
lesson is that admission control cannot be a queue length when the resource is
reclaimed after admission. Its own phase, or phase 4 — the placement is a
decision.

### Multi-tenant fairness — NOT grounded, do not author yet

No lab has more than one customer. Every scale target describes aggregate load,
so no design is ever punished for letting one tenant consume the whole budget.
`1/6` rations a shared budget but does not attribute it.

This needs a real reported behaviour before it becomes a spec: a documented
per-tenant limit, or a published incident where one tenant starved the rest.
Find that first, or drop the idea.

### Deletion that must reach every derived copy — NOT grounded, do not author yet

No lab deletes anything. Records enter, are corrected, expire by policy, and
are read back. None must be provably gone from a cache, a search index, a
materialised view and a replica, with the system able to say when it finished.

Same rule: find the documented behaviour or the published obligation first.
A regulation is a fact about why a requirement exists, never a thing to learn
— see `CLAUDE.md`, "The domain supplies the reasons, never the difficulty".

## Not gaps

Recorded decisions, listed so nobody proposes them twice.

- Consensus implementation, cloud-console navigation, and multi-cloud parity
  are out by `specs/01-systems-labs.md`, "Scope boundaries".
- Ten candidates were scored and cut in `docs/lab-selection.md`, among them a
  CRDT workspace, a Raft key-value store, an eBPF profiler and multi-region
  failover.
- `7/5` was cut for repeating `7/3`'s lesson, and phase 5 became `4/5`. Both
  gaps stay open on purpose.
