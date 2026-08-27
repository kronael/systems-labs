---
status: draft
---

# Standing offer service

## Brief

Design and build a standing offer service. The service holds an inventory of
goods and keeps them on offer in a marketplace it does not operate: the
marketplace accepts each offer, holds it standing, fills it or leaves it, and
streams every change of its own public state as numbered records. A supplied
rule states the terms each good should currently offer given that state; the
service's job is to keep its offers consistent with the rule, its inventory,
and the state the marketplace actually has.

An offer keeps acting after the service last touched it: it stands at the
marketplace and may fill at any moment, on terms the service chose from what
it knew at some earlier point. The service must never leave an offer standing
on knowledge it no longer has, and it must equally never fall silent and stay
off the marketplace once its knowledge has returned. Clients meanwhile read
the service's account of its inventory, its offers, and the marketplace as
last seen, and those answers must keep coming when the offering stops.

The required environment is the course-supplied marketplace, run locally,
whose contract is fixed. The concurrency model, the data layout, how the
service's account and its offers are brought back into agreement with the
marketplace, the process decomposition, and the coordination between the
service's components are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts the marketplace simulator, OpenTelemetry
collection, a metrics backend, the fault controller, and an optional second
application replica. The course also supplies generated types and clients for
the marketplace's interfaces, the supplied rule as a library with its
configuration, a deterministic seeded catalog and inventory, stream and fill
workloads with configured burst profiles, and the fault schedules behind
`make fault`.

The marketplace is prepared and outside the learner's processes. It keeps an
inspectable log of every offer accepted, revised, withdrawn, voided, and
filled, with the stream record number at which each happened. That log is the
observable boundary of the lab: where the service's account disagrees with
it, the marketplace is right.

The learner owns the application services, their persistence, and their
Compose layer. Standard Make targets start the environment, seed the catalog,
run feature tests, inject the failure schedule, run load, and capture
evidence. No cloud account is required.

## Requirements

The marketplace's contract is fixed. It streams every change of its public
state as consecutively numbered records, so a participant can always know
whether it has seen everything. It accepts, acknowledges, holds, and fills
offers, and withdraws them one at a time or all of a participant's at once,
within a session the participant must actively maintain; a session no longer
maintained is closed, and an action arriving on a closed session is refused,
never applied. Offers outlive the session that placed them, unless the
participant has registered, in advance, terms under which the marketplace
itself voids that participant's standing offers; registered terms take effect
a configured interval after the session closes, never immediately, and the
interval the marketplace accepts is bounded.

The service keeps its inventory on offer. While it can know it has seen
everything — the numbering contiguous, the session live, the process running
— at least the configured fraction of the catalog stands on offer, and every
standing offer's terms are what the supplied rule gives for a state the
service held no further behind the newest delivered record than the
configured freshness bound.

No offer stands on knowledge the service no longer has. From the record at
which the service can no longer know it has seen everything, every offer
still standing is unjustified, and a fill of an unjustified offer is a
violation wherever it appears — in the marketplace's log during the run or in
the reconciliation after it. The service must be able to state, for every
fill and every action in the run, the knowledge that justified it.

The service returns. From the record at which it can again know it has seen
everything, it must be offering again within the configured resumption bound,
counted in records, from an account that agrees with the marketplace's log.
Staying withdrawn is the same failure as staying on.

Answers outlive offers. Clients read the service's account — inventory held,
offers standing, the marketplace as last seen — at all times, including while
the service is not offering. Such an answer is served, not refused, and it
states what it is true of: the newest record it reflects and whether the
service currently knows it has seen everything. The freshness an answer
claims must agree with the recorded history, and a configured bound limits
how old a served answer may grow before it too is refused.

Inventory is never promised twice. Every fill commits exactly the units its
offer named, every fill the marketplace reports is honored in the service's
account exactly once, and units filled while the service was not running are
not offered again. The service survives restart: its account of inventory,
standing offers, and fills is rebuilt to agreement with the marketplace's
log.

Refusals and lapses surface. An action the marketplace refuses, a session
that closes, a gap the numbering proves — each maps to a visible outcome in
the service's responses or queryable status, never only to a log line.
Configuration comes from the standard TOML contract.

The scale target is a catalog of 10,000 goods holding one million units, a
stream sustained at 5,000 records per second with bursts to 20,000, 10,000
standing offers under a marketplace that accepts at most 500 offer actions
per second from one participant, fills arriving at up to 50 per second, a
coverage fraction of nine goods in ten, a freshness bound of 5,000 records, a
resumption bound of 25,000 records, and ten million records per evidence run.
These numbers size the problem; they are not pass thresholds. Latency and
coverage are judged against the configured bounds and the learner's declared
service level, not a fixed number.

The evidence must state, for every stretch of the run, whether the service
was offering or withdrawn and on what grounds; the coverage held while it
could know it had seen everything; the distance, in records, from each return
of knowledge to the first offer standing again; every action the marketplace
refused and what the service did next; and the offered stream rate against
the rate actually absorbed through the bursts. The fill and agreement checks
are recomputed from the marketplace's log, never from the service's account.
How the measurement is produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what knowledge justifies an offer, where that fact lives, and how the
  design can state it for any standing offer at any moment;
- which observations end the service's claim to have seen everything, which
  do not, and what each rule costs when it is wrong in either direction;
- what the service can know about its own standing offers while it cannot
  reach the marketplace, and what its design relies on during that time;
- what must have been observed before the first offer stands again after a
  lapse, and what that costs in records at the stream's burst rate;
- what an answer served while the service is not offering is true of, and how
  its claimed freshness is kept honest;
- how the service's account and the marketplace's log are kept in agreement
  across restart, reconnection, and fills that happened in between;
- what holding the coverage requirement costs while knowledge comes and goes
  — measured, not asserted.

Two further questions presuppose part of a design and are solution-bearing;
they publish to `HINTS.md`, never to the task:

- why a freshness check made immediately before an action protects nothing,
  and which side of the session must make the late action harmless;
- why the process whose knowledge is in doubt cannot be the only party able
  to void what it left standing, and what must already be arranged before the
  doubt begins.

The document must compare at least two viable designs without turning a known
product's name into the argument, and state one residual limitation.

## Adversarial evaluation

The failure schedule fires at named records and acknowledgements, never on a
timer. It suppresses the record after a named stream record and delivers the
ones that follow, leaving the session healthy, then fills a named offer still
standing; from the suppressed record until the service has again seen
everything, every fill of a standing offer is a violation. It severs the
session at the exact moment a named offer is acknowledged as standing and
holds it severed longer than the largest interval the marketplace accepts in
registered terms; that offer must be void once the terms take effect, a fill
of it afterwards is a violation, and a design that registered no terms fails
here by construction. It delivers a named record whose consequence under the
supplied rule is a revision, pauses the service's process before that
revision reaches the marketplace, holds the pause until the session has
closed, and then resumes the process; the late action must not take effect,
and any action applied after the pause whose justifying knowledge predates it
is a violation. After each of these it heals the stream at a named record;
the first offer must be standing again within the configured resumption
bound, from an account that agrees with the marketplace's log, and a design
that stays withdrawn fails here. Throughout every one of these stretches it
reads the service's account and requires answers that state truthfully what
they are true of. Finally it kills the service's process outright while
offers stand and fills arrive, then restarts it; afterwards no unit is
promised twice and every fill reported while the service was down is honored
exactly once.

At the end the marketplace's log, the stream, and the service's account are
read together. No fill in the recorded history landed on an offer that was
unjustified at that record, every reported fill is honored exactly once, no
unit was promised twice, every answer's claimed freshness agrees with the
records the service had actually seen, and the offering resumed within the
bound after every heal.

Checks observe only the marketplace's log and stream, the service's public
interface, process lifecycle, metrics, and the submitted evidence. They do
not require a particular internal structure, persistence choice, or
coordination primitive.

## Acceptance evidence

No fill lands on an unjustified offer at any point in the recorded history,
established from the marketplace's log and the stream record numbers rather
than from the service's account. Every reported fill is honored exactly once,
no unit is promised twice, and after every restart and reconnection the
service's account agrees with the marketplace's log. The configured coverage
holds while the service can know it has seen everything, the offering resumes
within the configured bound after every return of knowledge, and answers keep
coming throughout, each stating truthfully what it is true of. Refused
actions and lapses are returned or queryable, never only logged.

The report includes the exact offer, fill, and answer histories; every
stretch of withdrawal with its grounds and its length in records; coverage
over the run; the resumption distance after each lapse and its worst case;
every refused action and what followed it; the offered stream rate against
the absorbed rate through the bursts; and latency against the declared
service level. It names what the design gives up to hold both directions of
the requirement, and one residual limitation.

## Neighbouring systems

A practitioner might have reached for one of these instead. Their names
publish into `README.md`; here every documentation link is itself
solution-bearing, because each page states its position at exactly this lab's
boundary, so the links publish into `HINTS.md` with the boundary differences
rather than into `README.md`.

- **CME Globex** — [documentation](https://cmegroupclientsite.atlassian.net/wiki/display/EPICSANDBOX/Cancel+on+Disconnect).
  The venue side of this product's problem, solved as a registered facility:
  when the venue detects a session lost, it voids that participant's resting
  orders itself, exempting the kinds explicitly marked to outlive the
  session, so the participant's own doubt never has to run — the counterparty
  was armed in advance.
- **Kubernetes** — [documentation](https://kubernetes.io/docs/concepts/architecture/nodes/).
  Its node controller acts on remote liveness evidence, and when that
  evidence implies every node failed at once it concludes the evidence itself
  is broken and stops evicting; the self-distrust suspends future authority
  but never voids anything already done.
- **Recursive DNS resolvers** — [documentation](https://www.rfc-editor.org/rfc/rfc8767.txt).
  Keep answering from expired data when the authority cannot be reached,
  bounded on the order of days, because an answer holds no standing authority
  once given and expires on its own — the opposite rule, for the opposite
  kind of output.

The lab does not run them.

## Scope

The expected focused time is twelve to eighteen hours. The marketplace, the
supplied rule, the catalog and inventory, the workloads, telemetry, and
faults are prepared. No store is required: the state that must survive
restart is the service's own account, and how it is kept is the data-layout
decision the lab judges. A simple design fails this lab at its scale target:
one process that takes the stream, revises offers, keeps its account, and
maintains the session in a single line of work holds together while the
stream is quiet, but through a 20,000-record burst its intake falls past the
freshness bound while its session upkeep starves, so its offers stand
unjustified exactly when fills keep arriving; and a design that treats every
doubt as total, voiding and re-placing all 10,000 offers each time, pays the
500-action ceiling in full for every wobble and finishes the run far below
the coverage a steadier design holds. Choosing offer terms is outside the
problem — the rule is supplied. So are bounding the service's own action
volume, competing against other participants, running against more than one
marketplace, and payments.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  workload, fault, and evidence contracts.
- [SEC Release 34-70694](https://www.sec.gov/litigation/admin/2013/34-70694.pdf)
  — the origination. Knight Capital "did not have procedures in place to halt
  SMARS's operations in response to its own aberrant activity" and "did not
  have a mechanism to test whether their systems were relying on stale data";
  the 2011 incident the order records was remediated by "changing the control
  so that this system would stop providing quotes after receiving an
  execution". Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- [Binance, how to manage a local order book correctly](https://raw.githubusercontent.com/binance/binance-spot-api-docs/master/web-socket-streams.md)
  — the gap barrier's origination. The update procedure opens: "If the event
  first update ID (`U`) is greater than the update ID of your local order
  book + 1, you have missed some events. Discard your local order book and
  restart the process from the beginning." A missed record invalidates the
  whole derived view, not the record. Solution-bearing: this belongs in
  `HINTS.md`, never in `README.md`.
- [CME Globex Cancel on Disconnect](https://cmegroupclientsite.atlassian.net/wiki/display/EPICSANDBOX/Cancel+on+Disconnect)
  — "If a lost connection is detected, COD cancels all resting futures and
  options orders for the disconnected registered iLink user", excluding GTC
  and GTD orders; a connection that answers no test request within a
  heartbeat interval "is assumed to be stale and the socket is closed".
  Solution-bearing.
- [Deribit Cancel On Disconnect](https://docs.deribit.com/) — "all orders
  created via this connection will be automatically cancelled when the
  connection is closed", while a graceful logout cancels nothing: the void is
  for doubt, not for departure. Solution-bearing.
- [Bybit Disconnect Cancel All](https://bybit-exchange.github.io/docs/v5/order/dcp)
  — the protection window is configured and bounded, "[3, 300], unit:
  second", and fires when the client has not reconnected and resumed
  heartbeats within it. Solution-bearing.
- [Kleppmann, How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
  — "You cannot fix this problem by inserting a check on the lock expiry just
  before writing back to storage", because "GC can pause a running thread at
  any point, including the point that is maximally inconvenient for you"; the
  side holding the resource checks "a fencing token", "simply a number that
  increases … every time a client acquires the lock". Solution-bearing.
- [RFC 8767](https://www.rfc-editor.org/rfc/rfc8767.txt) — serving stale DNS
  data "under the metaphorical assumption that 'stale bread is better than no
  bread'", with staleness "capped on the order of days to weeks, with a
  recommended cap of 604,800 seconds (7 days)". Solution-bearing.
- [RFC 9309 §2.3.1.4](https://www.rfc-editor.org/rfc/rfc9309.txt) — "If the
  robots.txt file is unreachable due to server or network errors … the
  crawler MUST assume complete disallow", with an escape hatch only after "a
  reasonably long period of time (for example, 30 days)". Solution-bearing.
- [Kubernetes node controller](https://kubernetes.io/docs/concepts/architecture/nodes/)
  — when all zones are completely unhealthy, "the node controller assumes
  that there is some problem with connectivity between the control plane and
  the nodes, and doesn't perform any evictions". Solution-bearing.
- [AWS DynamoDB service disruption, 2015-09-20](https://aws.amazon.com/message/5467D2/)
  — storage servers unable to confirm their membership "temporarily
  disqualify themselves from accepting requests", and that self-
  disqualification turned a network blip into a region-wide outage: the
  counter-case that makes the resumption requirement a gate. Solution-bearing.
- [Amazon Builders' Library, static stability](https://aws.amazon.com/builders-library/static-stability-using-availability-zones/)
  — "the data plane maintains its existing state and continues working even
  in the face of a control plane impairment": the counter-case for the
  answers this service must keep serving. Solution-bearing.
- [RIPE-378](https://www.ripe.net/publications/docs/ripe-378/) — route-flap
  damping deployed and then recommended against, because "a simple prefix
  withdrawal can result in the appearance of a major flap event a few AS hops
  away", suppressing perfectly valid routes: withdrawal itself can cascade.
  Solution-bearing.
- Implementation pointers do not exist while the spec is `draft`.
