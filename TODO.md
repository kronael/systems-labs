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

It is its own lab and not an extension of `4/5` or `7/6`: each already carries
one falsified model and sits at the top of its hour budget, and a barrier added
to either could be left unreached by a correct design.

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

## Not gaps

Recorded decisions, listed so nobody proposes them twice.

- Consensus implementation, cloud-console navigation, and multi-cloud parity
  are out by `docs/contract.md`, "Scope boundaries".
- Ten candidates were scored and cut in `docs/lab-selection.md`, among them a
  CRDT workspace, a Raft key-value store, an eBPF profiler and multi-region
  failover.
- `7/5` was cut for repeating `7/3`'s lesson, and phase 5 became `4/5`. Both
  gaps stay open on purpose.
- A deletion lab: `4/3` and `8/3` carry it as faults. `1/3` cannot: a tombstone
  needs a per-event compacted topic, and that lab keys by order, so the fault
  would never activate.
- A multi-tenant fairness lab: `1/8` and `4/1` carry it as a requirement. `2/1`
  cannot: the platform refuses before the learner's code runs and knows nothing
  about accounts.
- A backwards-clock lab: `1/1` and `1/2` carry it as a fault.
- Accepted work discarded to make room is `1/8`.
