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
it knew at some earlier point. A standing offer is therefore a continuing
action, justified only while the service can still keep it consistent with
the rule — a current view of the marketplace, and the standing power to
revise or withdraw what it placed. The service must never let an offer
outlive that justification, and it must equally never fall silent and stay
off the marketplace once view and power have returned. Clients meanwhile read
the service's account of its inventory, its offers, and the marketplace as
last seen, and those answers must keep coming when the offering stops.

The required environment is the course-supplied marketplace, run locally,
whose contract is fixed. The concurrency model, the data layout, how the
service's account and its offers are brought back into agreement with the
marketplace, the process decomposition, and the coordination between the
service's components are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts the marketplace simulator, OpenTelemetry
collection, a metrics backend, and the fault controller. The course also
supplies generated types and clients for the marketplace's interfaces, the
supplied rule as a library with its configuration, a deterministic seeded
catalog and inventory, stream and fill workloads with configured burst
profiles, and the fault schedules behind `make fault`.

The marketplace is prepared and outside the learner's processes. It keeps an
inspectable log of every offer accepted, revised, withdrawn, voided, and
filled, every action refused with its reason, every session opened and
closed, and every rejoin served with the record its read was taken at — each
with the stream record number at which it happened. That log is the record of
the run, read in the reconciliation after it and never a surface the service
can consume while it runs: where the service's account disagrees with it, the
marketplace is right.

The learner owns the application services, their persistence, and their
Compose layer. Standard Make targets start the environment, seed the catalog,
run feature tests, inject the failure schedule, run load, and capture
evidence. No cloud account is required.

## Requirements

The marketplace's contract is fixed. It streams every change of its public
state as consecutively numbered records, delivered once, at the edge: a
record not taken at delivery is never re-delivered, and a participant that
rejoins the stream is handed a complete read of the marketplace's public
state as of a named record, with delivery continuing from that record's
successor. The stream keeps a stated cadence — it is never silent longer than
the interval the contract names — so a participant can always know, from
numbering and cadence together, whether it has seen everything. The
marketplace accepts, acknowledges, or refuses each action in the order sent,
within a session the participant must actively maintain; the stream and the
session are separate channels, and losing one proves nothing about the other.
A session no longer maintained is closed within the contract's maintenance
deadline, and an action arriving on a closed session is refused, never
applied. Maintaining the session, rejoining the stream, and reading the
marketplace's state draw nothing from its action ceiling; the ceiling covers
offer actions alone. Offers outlive the session that placed them, unless the
participant has registered, in advance, terms under which the marketplace
itself voids that participant's standing offers; registered terms take effect
a configured interval after the session closes, never immediately, and the
interval the marketplace accepts is bounded.

The service keeps its inventory on offer. While it can know it has seen
everything and can act — the delivered numbering contiguous, no silence past
the stream's cadence, the session live, its processes running — at least the
configured fraction of the goods with units remaining stands on offer, and
every standing offer's terms are what the supplied rule gives for a state no
further behind the newest record the marketplace has emitted than the
freshness bound. The one allowance is restoration: after each return point,
coverage must be back at the fraction within the restoration bound and hold
until the next lapse. A service standing below the fraction while it could
know and could act has failed this requirement as surely as a service that
never withdraws fails the next one.

No offer outlives its justification. A standing offer's justification is a
record: a delivered record, inside the freshness bound, whose state under the
supplied rule gives exactly the offer's terms — held while the numbering is
contiguous, the cadence unbroken, and the service retains the power to revise
or withdraw. The service's account must state that record, by identity, for
every standing offer, every action it took, and every fill the marketplace
reports.

Liability is dated from what the service could observe and could act on,
never from what merely happened. A fill of a standing offer is a violation
only where, for at least the reaction allowance before it, counted in records
the marketplace emitted, three things had held together: the offer was
unjustified, the lapse was provable at the service's boundary — a delivered
number that does not follow its predecessor, silence past the cadence, terms
left past the freshness bound — and the service held the power to act, a live
session and running processes. Every other fill of an unjustified offer is
exposure, not violation: bought by the declared allowance, by the registered
interval while its countdown runs after a close, or by the maintenance
deadline while the service's processes were held stopped; the evidence must
name every such fill and the window that admitted it. Exposure is bounded by
arrangement, and the bound is enforced: once registered terms are due,
nothing of the participant's may stand, and a fill after the due point is a
violation wherever it appears — in the marketplace's log during the run or in
the reconciliation after it. A design that registered no terms bought
unbounded exposure; its liability begins one largest-acceptable interval
after its session closed.

The service returns. Every lapse has a return point on the public record,
written to the run manifest: the record that proves a gap, the record at
which a severed session may again be established, the record at which cut
delivery is restored, the first record the marketplace emits after stopped
processes resume or killed ones are restarted. From each return point the
first offer must be standing again within the resumption bound and coverage
restored within the restoration bound, both counted in emitted records, from
an account that agrees with the marketplace's log. Staying withdrawn is the
same failure as staying on.

Answers outlive offers. Clients read the service's account — inventory held,
offers standing, the marketplace as last seen — at all times, including while
the service is not offering. Such an answer is served, not refused, and it
states what it is true of: the newest record it reflects and whether the
service currently knows it has seen everything. An answer is honest when its
content agrees with the marketplace's log at the record it claims and that
record is no newer than the newest the marketplace had emitted. The answer
age — how far, in emitted records, the claimed record may lag the newest
emitted — is bounded: an answer older than the declared age is refused, and
the refusal states why.

Inventory is never promised twice. Every fill commits exactly the units its
offer named, every fill the marketplace reports is honored in the service's
account exactly once — by the fill's identity, never by totals alone — and
units filled while the service was not running are not offered again. The
service survives restart: its account of inventory, standing offers, and
fills is rebuilt to agreement with the marketplace's log.

Refusals and lapses surface. An action the marketplace refuses, a session
that closes, a gap the numbering proves, a silence past the cadence — each
maps to a visible outcome in the service's responses or queryable status,
never only to a log line. Configuration comes from the standard TOML
contract.

The scale target is a catalog of 10,000 goods holding one million units, a
stream sustained at 5,000 records per second with bursts to 20,000 held for
up to 50,000 records, terms changing for at most forty goods per thousand
records, 10,000 standing offers under a marketplace that accepts at most 500
offer actions per second from one participant, fills arriving at up to 50 per
second, a coverage fraction of nine goods in ten, a freshness bound of 5,000
records, a resumption bound of 25,000 records, a restoration bound of
150,000 records, and ten million records per evidence run. These numbers size
the problem; they are not pass thresholds. The coverage fraction is
configured and dimensionless. Every record-counted bound — freshness,
reaction allowance, resumption, restoration, answer age — is a component of
the service level the learner declares and records in the report. Bounds
named above are judged against those values as the calibration reference;
the reaction allowance and the answer age carry no course value and are
defended against the run's own observed histories. A looser declaration is
not itself a failure, but it widens the exposure the evidence must account
for, fill by fill, and the report must defend it. Latency and coverage are
judged against the configured fraction and the declared service level, never
a fixed number.

The evidence must state, for every stretch of the run, whether the service
was offering or withdrawn and on what grounds; the coverage held, over goods
with units remaining, while it could know and act; the distance, in emitted
records, from each return point to the first offer standing and to coverage
restored; every fill named as exposure with the window that admitted it;
every action the marketplace refused and what the service did next; the
declared service level against the calibration reference; and the offered
stream rate against the rate actually absorbed through the bursts. Every
gating quantity — fills, justification, agreement, coverage, the resumption
and restoration distances, the withdrawal stretches — is recomputed from the
marketplace's log and the run manifest, never from the service's account; the
absorbed rate alone comes from the service's metrics and gates nothing. How
the tradeoff measurements — rates, latency, the cost of holding both
directions — are produced is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- where the justification of every standing offer lives, and how the design
  can state it — the record it rests on — for any offer, action, or fill at
  any moment;
- which observations end neither the service's view nor its power to act, and
  what a rule wrong in either direction costs when it fires;
- what the service can know about its own standing offers while it cannot
  reach the marketplace, and what its design relies on during that time;
- what must have been observed before the first offer stands again after a
  lapse, and what that costs in records at the stream's burst rate;
- what an answer served while the service is not offering is true of, and how
  its claimed freshness is kept honest;
- how the service's account and the marketplace's log are kept in agreement
  across restart, reconnection, and fills that happened in between;
- how each declared bound — reaction allowance, freshness, resumption,
  restoration, answer age — was chosen, and what the declaration buys and
  what it exposes;
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
timer, and `make fault` writes what it fired into the run manifest under
`evidence/`: the suppressed, proving, lifting, healing, and return records,
the named offers and fills, and a transcript of every answer it read with the
newest record the marketplace had emitted at that moment. Checks select on
the manifest and the marketplace's log, never on literals, and never on the
service's account.

It suppresses the record after a named stream record and delivers the ones
that follow, leaving the session healthy; when delivery reaches the first
record past the proof record plus the declared reaction allowance, it fills a
named offer if that offer still stands on its pre-gap justification. That
fill is a violation; fills landing before the allowance's edge are exposure
the evidence must name. What must hold after: from the proof record, the
first offer within the resumption bound and coverage within the restoration
bound, from an account that agrees with the log.

It repeats the suppression as a run of single-record gaps at named records,
spaced closer together than the full catalog could be re-placed under the
action ceiling. Coverage across the stretch is judged at the configured
fraction: the design pays for what each gap actually cost it, and a design
that buys every gap with a total void spends the stretch below the fraction
and fails here.

It severs the session at the exact moment a named offer is acknowledged as
standing, leaves the stream flowing, and holds the severance past the largest
interval the marketplace accepts in registered terms, lifting it at a named
record after the void is due. The checks here are orderings in the log, never
record literals: the close, then the void exactly when the registered terms
come due, and no fill of the named offer after the due point. Fills inside
the countdown are the registered interval's exposure, named in the evidence.
A design that registered no terms leaves the offer standing, and the schedule
fills it one largest-acceptable interval after the close — the violation that
design bought. What must hold after the lift: a session re-established, the
voided inventory offered again, coverage within the restoration bound.

In the same instant it delivers a named record whose consequence under the
supplied rule changes a named offer's terms, it stops every learner process,
so nothing that record provokes can reach the marketplace; while the pause
holds, the stream moves on, a further named record changes that offer's terms
again, and the session closes. At a named record after the close it resumes
the processes. The late action must not take effect — the closed session
refuses it — and any action applied after the pause whose justifying record
predates it is a violation the moved terms make visible. While every learner
process is held stopped, and only then, the answers requirement is suspended;
the first answer after resumption states truthfully what it is true of.

It cuts the stream's delivery outright at a named record and refuses every
rejoin until it heals delivery at a named record, holding the cut longer, in
emitted records, than the declared answer age; the session stays live
throughout. Answers must keep coming through the cut, aging honestly, until
the declared age refuses them with the grounds stated. When the silence has
outlasted the cadence and the declared allowance, it fills a named offer if
that offer still stands — a violation if it does. From the heal, the return
is checked within the declared bounds.

When a named fill is appended to the marketplace's log, it kills every
learner process outright, then restarts them at a named record. Afterwards no
unit is promised twice, every fill reported while the service was down is
honored exactly once by identity, and the account is rebuilt to agreement
with the log.

Inside every one of these stretches the workload moves the state of at least
one good the schedule later asks about, so an answer claiming sight of it is
checkable against the log; throughout every stretch the schedule reads the
service's account and requires answers that state truthfully what they are
true of.

At the end the marketplace's log, the manifest's delivery timeline, and the
service's account are read together. Justification is recomputed for every
fill from the log, the timeline, and the declared allowance: no fill is a
violation under the liability rule, every fill inside a declared or
registered window is named as exposure, every reported fill is honored
exactly once by identity, no unit was promised twice, every answer in the
transcript agrees with the log at the record it claims, and the offering and
the coverage returned within the declared bounds after every return point.

Checks observe only the marketplace's log and stream, the service's public
interface, process lifecycle, metrics, the run manifest, and the submitted
evidence. They do not require a particular internal structure, persistence
choice, or coordination primitive.

## Acceptance evidence

No fill in the recorded history is a violation under the liability rule —
justification recomputed from the marketplace's log, the manifest's delivery
timeline, and the declared allowance, never from the service's account — and
every fill inside a declared or registered window is named as exposure with
the window that admitted it. Every reported fill is honored exactly once, by
identity; no unit is promised twice; after every restart and reconnection the
service's account agrees with the marketplace's log. The configured coverage
holds over goods with units remaining while the service could know and could
act, the offering and the coverage return within the declared bounds after
every return point, and answers keep coming throughout — outside the
stretches in which the schedule holds every learner process stopped — each
agreeing with the log at the record it claims. Refused actions and lapses are
returned or queryable, never only logged, and the log's own refusal record
confirms the service's.

The report includes the exact offer, fill, and answer histories; every
stretch of withdrawal with its grounds and its length in records; coverage
over the run and per stretch; the resumption and restoration distances after
each lapse and their worst cases; the exposure account, fill by fill; every
refused action and what followed it; the offered stream rate against the
absorbed rate through the bursts; the declared service level against the
calibration reference, with every looser declaration defended; and latency
against the declared service level. It names what the design gives up to
hold both directions of the requirement, and one residual limitation.

## Neighbouring systems

A practitioner would not have built this product bare-handed against a live
marketplace, and that is exactly the trouble with naming what they would have
reached for: at this lab's boundary every genuine neighbour — the protections
serious venues document on the marketplace's own side, the off-the-shelf
frameworks that keep standing offers current — carries the lab's answer in
its name or on its landing page. Naming a neighbour must orient without
solving, and here no candidate does the first without the second: each of the
three below is one search away from the whole design. This lab's `README.md`
therefore names no neighbouring system — a deliberate, recorded deviation
from the landscape contract — and all three entries publish, names, links,
and boundary differences together, into `HINTS.md` only.

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

The expected focused time is sixteen to twenty-four hours: the
falsify-and-rebuild loop between the gates is the teaching, and the build
surface — the account and its durability, the action economy, the honest
answers, and the justification the account must state — is the widest in the
phase. The marketplace, the supplied rule, the catalog and inventory, the
workloads, telemetry, and faults are prepared. No store is required: the
state that must survive restart is the service's own account, and how it is
kept is the data-layout decision the lab judges. A simple design fails this
lab at its scale target, and both failure narratives that follow are
solution-bearing: they publish into `HINTS.md`, never into `README.md`. One
process that takes the stream, revises offers, keeps its account, and
maintains the session in a single line of work holds together while the
stream is quiet, but through a burst the term churn alone outruns the action
ceiling while its intake falls past the freshness bound and its session
upkeep starves, so its offers stand unjustified exactly when fills keep
arriving. And a design that treats every wobble — a refusal, a burst, a slow
answer — as lost knowledge, voiding and re-placing all 10,000 offers each
time, pays the 500-action ceiling in full for every storm it starts and
spends the small-gap stretch and its own self-made doubts below the coverage
fraction while, on this spec's own definition, it could know and could act:
the coverage requirement convicts it directly. Choosing offer terms is
outside the problem — the rule is supplied. So are bounding the service's own
action volume, competing against other participants, running against more
than one marketplace, and payments.

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
