# TODO

Author-facing. `BUGS.md` holds what is wrong with what exists; this file holds
what does not exist yet. A proposal moves out of here by becoming a lab under
`labs/<phase>/`, and it may not become one until its quirk is grounded — see
`CONTRIBUTING.md`, "Adding a lab", step 1.

## The scaffold

Nothing is built. `docs/shared-scaffold.md` specifies the generator, the
fault controller, the evidence writer and `template/`, and the approval
boundary blocks all of it while `docs/contract.md` is `draft`. The
controller's three layers — transport, process, and clock — are specified now,
and every lab that depends on one names it, but none of it is built. That work
comes before any new lab.

## Gaps in the catalog

Each entry below names a failure no existing lab reaches. The first three
carry a documented, publicly reported behaviour, fetched and quoted. The last
two do not, and must not be authored until they do.

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

Two studies were run against existing labs on 2026-09-20 and both were
rejected: `4/5` and `7/6` each already carry one falsified model and sit at
the top of their hour budgets, and each proposed barrier could be left
unreached by a correct design. What the studies produced belongs to this lab
instead.

- The envelope does not carry the contract's version. CloudEvents defines
  `specversion` as "The version of the CloudEvents specification which the
  event uses", and producers "MUST use a value of `1.0`", so it reads the
  same on both contract versions and a receiver keying on it sees no change.
  The primer states that the specification "does not mandate any particular
  pattern to be used, or even the use or consideration of versioning at all",
  and that a backwards-incompatible data change should generally change the
  `type`, with the producer "encouraged to produce both the old event and the
  new event for some time".
- The two compute models make the skew asymmetric by their own documents. A
  Kubernetes rolling update keeps both replica sets serving, and "Each
  rollback updates the revision of the Deployment". A published Lambda
  version is immutable, an unqualified call reaches `$LATEST`, which "any
  time you deploy your function code, you overwrite", and a weighted alias
  points at "a maximum of two Lambda function versions" and splits between
  them probabilistically.
- Where a lab holds the traffic share, it needs a supplier: no vendor page
  ties alias weights to an event source mapping, so the share is a declared
  model on a layer the fault contract does not yet name. That gap is the
  lab's to state, not to assume.

A second candidate sits beside it and should stay separate: continuous
asynchronous rollout, where nobody can force a client to upgrade. Its facts
are that a fetched bundle can never change because "Any difference in the
content will produce a different CID", that a resolved name is remembered for
the record's own TTL hint, and that a program upgrade replaces the bytes for
every caller with a documented visibility delay. The asymmetry is the lesson:
one party moves once for everyone, the other moves once per user.

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

## Closed

- **Time that goes backwards** became a fault rather than a lab. `1/1` steps
  the clock backwards inside the comparison that decides whether a quote is
  still valid, and `1/2` inside the one that decides whether an
  acknowledgement window has passed. Commit `8a02696`.
- **Accepted work discarded to make room** became
  `labs/1/8-streaming-completion-service`, in phase 1 because the learner
  operates the engine and the service in front of it, and beside `1/1`
  because the contrast is the lesson: `1/1` refuses before it admits, `1/8`
  has the resource taken back after it admitted.

## Not gaps

Recorded decisions, listed so nobody proposes them twice.

- Consensus implementation, cloud-console navigation, and multi-cloud parity
  are out by `docs/contract.md`, "Scope boundaries".
- Ten candidates were scored and cut in `docs/lab-selection.md`, among them a
  CRDT workspace, a Raft key-value store, an eBPF profiler and multi-region
  failover.
- `7/5` was cut for repeating `7/3`'s lesson, and phase 5 became `4/5`. Both
  gaps stay open on purpose.
