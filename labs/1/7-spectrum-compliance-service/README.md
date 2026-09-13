# Spectrum compliance service

Design and build a spectrum compliance service. The service operates a fleet
of radio transmitters and keeps them legally on the air under a spectrum
coordinator it does not run: the coordinator authorizes each transmitter's
channel and power, may suspend, modify, or revoke any authorization at any
moment, and streams every change of its own public state as numbered records.
A supplied rule states the channel and power each transmitter should
currently hold given that state; the service's job is to keep the fleet
transmitting consistently with the rule, its authorizations, and the state
the coordinator actually has.

A transmitter keeps acting after the service last touched it: it stays on the
air as last set, and its transmissions keep landing, on an assignment the
service chose from what it knew at some earlier point. An on-air transmitter
is therefore a continuing action, justified only while the service can still
keep it consistent with the rule — a current view of the coordinator's state,
a standing authorization, and the power to act on what it set transmitting.
The service must never let a transmitter outlive that justification, and it
must equally never fall silent and leave the fleet dark once view and
authorization have returned. Clients meanwhile read the service's account of
its fleet, its authorizations, and the coordinator as last seen, and those
answers must keep coming when the transmitting stops.

The setting is modeled on shared-spectrum regimes such as CBRS, in which a
coordinator admits ordinary transmitters to a band whose first holders —
incumbents — may return at any moment; when one does, the coordinator
suspends or moves the authorizations in the way, and a transmitter must
comply within a mandated deadline. No radio engineering enters the lab:
channel, power, and location are numbers the coordinator's records give
meaning to, and propagation, antennas, and interference physics are outside
the problem.

The required environment is the course-supplied coordinator and fleet, run
locally, whose contracts are fixed. The concurrency model, the data layout,
how the service's account, its authorizations, and its fleet are brought back
into agreement with the coordinator, the process decomposition, and the
coordination between the service's components are the learner's decisions.

## What you are given

The supplied Compose stack starts the coordinator simulator with its band
monitor, the transmitter fleet, OpenTelemetry collection, a metrics backend,
and the fault controller. The course also supplies generated types and
clients for the coordinator's and the transmitters' interfaces, the
assignment rule as a library with its configuration, a deterministic seeded
fleet and channel plan, stream and traffic workloads with configured burst
profiles, and the fault schedules behind `make fault`.

The coordinator and the fleet are prepared and outside the learner's
processes. The coordinator keeps an inspectable log of every authorization
granted, renewed, modified, suspended, reinstated, lapsed, revoked, and
relinquished, every action refused with its reason, every session opened and
closed, every rejoin served with the record its read was taken at, and —
through its monitor — every transmission observed on the air, each with the
stream record number at which it happened. That log is the record of the run,
read in the reconciliation after it and never a surface the service can
consume while it runs: where the service's account disagrees with it, the
coordinator is right.

The learner owns the application services, their persistence, and their
Compose layer. Standard Make targets start the environment, seed the fleet,
run feature tests, inject the failure schedule, run load, and capture
evidence. No cloud account is required.

## Requirements

The coordinator's contract is fixed. It streams every change of its public
state — authorizations and their standing, channel availability, incumbent
activity, sessions, and every transmission its monitor observes on the air —
as consecutively numbered records, delivered once, at the edge: a record not
taken at delivery is never re-delivered, and a participant that rejoins the
stream is handed a complete read of the coordinator's public state as of a
named record, with delivery continuing from that record's successor. The
stream keeps a stated cadence — it is never silent longer than the interval
the contract names — so a participant can always know, from numbering and
cadence together, whether it has seen everything. The coordinator accepts,
acknowledges, or refuses each action in the order sent, within a session the
participant must actively maintain; the stream and the session are separate
channels, and losing one proves nothing about the other — an acknowledgement
accepts an action, and what the action changed appears only on the stream. A
session no longer maintained is closed within the contract's maintenance
deadline, and an action arriving on a closed session is refused, never
applied. Maintaining the session, rejoining the stream, and reading the
coordinator's state draw nothing from its action ceiling; the ceiling covers
authorization actions alone — requesting, renewing, modifying, relinquishing.
Every acknowledgement that grants or renews an authorization names its
expiry, never farther ahead than the horizon the contract names: unrenewed
past the expiry, the authorization covers no transmission, and its lapse goes
on the public record. Authorizations outlive the session that obtained them —
a close voids nothing, and an unrenewed authorization simply runs out. The
coordinator may suspend an authorization at any moment and later reinstate
it, both public records; a suspended authorization covers no transmission,
stays renewable, and returns intact at reinstatement. An authorization
unrenewed past the contract's revocation period is revoked, and what it
covered must be asked for again.

The fleet's contract is fixed too. A transmitter transmits as the service
last set it — channel, power, on or off the air — and keeps doing so across
the service's silence, pause, and restart; its transmissions land whenever
traffic is offered while it stands on the air, and the coordinator's monitor
observes every one, authorized or not. Setting or silencing a transmitter
draws nothing from the coordinator's ceiling and needs no session; it needs
only the service's own processes running. A transmitter left on the air
therefore outlives everything except the service's decision to silence it —
unless the service has registered, in advance, a cutoff under which the
transmitter takes itself off the air when not refreshed; a registered cutoff
takes effect a configured interval after the service's last contact with that
transmitter, never immediately, and the interval the transmitter accepts is
bounded.

The service keeps its fleet on the air. While it can know it has seen
everything and can act — the delivered numbering contiguous, no silence past
the stream's cadence, the session live, its processes running — at least the
configured fraction of the transmitters to which the rule gives a current
assignment stands on the air, and every on-air transmitter holds exactly the
channel and power the rule gives for a state no further behind the newest
record the coordinator has emitted than the freshness bound, under an
authorization that state shows standing. The one allowance is restoration:
after each return point, coverage must be back at the fraction within the
restoration bound and hold until the next lapse. A fleet held dark below the
fraction while the service could know and could act has failed this
requirement as surely as a fleet still sounding past its justification fails
the next one.

No transmitter outlives its justification. An on-air transmitter's
justification is a record: a delivered record, inside the freshness bound,
whose state under the supplied rule gives exactly the transmitter's channel
and power and shows its authorization standing — not expired, not suspended,
not revoked — held while the numbering is contiguous, the cadence unbroken,
and the service retains the power to move or silence the transmitter. The
service's account must state that record, by identity, for every on-air
transmitter, every action it took, and every transmission the coordinator's
monitor reports.

Liability is dated from what the service could observe and could act on,
never from what merely happened. A transmission is a violation only where,
for at least the reaction allowance before it, counted in records the
coordinator emitted, three things had held together: the transmitter stood on
the air unjustified, the lapse was provable at the service's boundary — a
delivered number that does not follow its predecessor, silence past the
cadence, an assignment left past the freshness bound, a suspension or lapse
on the delivered record, a hole its own stopped intake proves — and the
service held the power to act, its processes running and the transmitter in
reach. Every other transmission of an unjustified transmitter is exposure,
not violation: bought by the declared allowance while a provable lapse was
fresh, or by a registered cutoff whose countdown was still running while the
service's processes were held stopped; the evidence must name every such
transmission and the window that admitted it. Two bounds are known before
they arrive, and both are enforced. An expiry comes named in the
acknowledgement that bought its authorization: a transmission past the expiry
while the service's processes ran is a violation with no allowance at all,
because nothing about that deadline was left to observe. A registered cutoff
is the transmitter's own: once its due point passes, that transmitter is off
the air, and a transmission after the due point is a violation wherever it
appears — in the coordinator's log during the run or in the reconciliation
after it. A design that registered no cutoff bought unbounded exposure; its
liability begins one largest-acceptable interval after its last contact with
each transmitter.

The service returns. Every lapse has a return point on the public record,
written to the run manifest: the record that proves a gap, the record at
which a severed session may again be established, the record at which cut
delivery is restored, the first record the coordinator emits after stopped
processes resume or killed ones are restarted. From each return point the
first transmitter must be back on the air within the resumption bound and
coverage restored within the restoration bound, both counted in emitted
records, from an account that agrees with the coordinator's log. Staying dark
is the same failure as staying on — and it compounds: an authorization
unrenewed through a long silence is revoked, and what would have been a
renewal becomes a fresh request, spent against the same ceiling.

Answers outlive the air. Clients read the service's account — the fleet as
set, the authorizations held, the coordinator as last seen — at all times,
including while the fleet is dark. Such an answer is served, not refused, and
it states what it is true of: the newest record it reflects and whether the
service currently knows it has seen everything. An answer is honest when its
content agrees with the coordinator's log at the record it claims and that
record is no newer than the newest the coordinator had emitted. The answer
age — how far, in emitted records, the claimed record may lag the newest
emitted — is bounded: an answer older than the declared age is refused, and
the refusal states why.

The account survives what the service did not see. Every transmission the
coordinator's monitor reports is honored in the service's account exactly
once — by the transmission's identity, never by totals alone — including
those observed while the service was not running. Every authorization is held
by exactly one transmitter and stated with the record that justifies it; an
authorization that lapsed, was suspended, or was revoked while the service
was down is not transmitted under again. The service survives restart: its
account of the fleet, the authorizations, and the observed transmissions is
rebuilt to agreement with the coordinator's log.

Refusals and lapses surface. An action the coordinator refuses, a session
that closes, a gap the numbering proves, a silence past the cadence — each
maps to a visible outcome in the service's responses or queryable status,
never only to a log line. Configuration comes from the standard TOML
contract.

The scale target is a fleet of 10,000 transmitters under one coordinator, a
stream sustained at 5,000 records per second with bursts to 20,000 held for
up to 50,000 records, assignments changing for at most forty transmitters per
thousand records, 10,000 standing authorizations whose expiries lie never
farther than 300 seconds ahead, a coordinator that accepts at most 500
authorization actions per second from one participant, traffic driving
observed transmissions at up to 50 per second, a coverage fraction of nine
transmitters in ten, a freshness bound of 5,000 records, a resumption bound
of 25,000 records, a restoration bound of 150,000 records, and ten million
records per evidence run. These numbers size the problem; they are not pass
thresholds. The coverage fraction is configured and dimensionless. Every
record-counted bound — freshness, reaction allowance, resumption,
restoration, answer age — is a component of the service level the learner
declares and records in the report. Bounds named above are judged against
those values as the calibration reference; the reaction allowance, the answer
age, and the registered cutoff interval carry no course value and are
defended against the run's own observed histories. A looser declaration is
not itself a failure, but it widens the exposure the evidence must account
for, transmission by transmission, and the report must defend it. Latency and
coverage are judged against the configured fraction and the declared service
level, never a fixed number.

The evidence must state, for every stretch of the run, whether the fleet was
on the air or dark and on what grounds; the coverage held, over transmitters
the rule assigned, while the service could know and act; the distance, in
emitted records, from each return point to the first transmitter on the air
and to coverage restored; every transmission named as exposure with the
window that admitted it; every action the coordinator refused and what the
service did next; the declared service level against the calibration
reference; and the offered stream rate against the rate actually absorbed
through the bursts. Every gating quantity — transmissions, justification,
agreement, coverage, the resumption and restoration distances, the dark
stretches — is recomputed from the coordinator's log and the run manifest,
never from the service's account; the absorbed rate alone comes from the
service's metrics and gates nothing. How the tradeoff measurements — rates,
latency, the cost of holding both directions — are produced is the learner's
choice.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- where the justification of every on-air transmitter lives, and how the
  design can state it — the record it rests on — for any transmitter, action,
  or transmission at any moment;
- which observations end neither the service's view nor its power to act, and
  what a rule wrong in either direction costs when it fires;
- what the service can know about its own fleet and authorizations while it
  cannot reach the coordinator, and what its design relies on during that
  time;
- what must have been observed before the first transmitter returns to the
  air after a lapse, and what that costs in records at the stream's burst
  rate;
- what an answer served while the fleet is dark is true of, and how its
  claimed freshness is kept honest;
- how the service's account and the coordinator's log are kept in agreement
  across restart, reconnection, and transmissions observed in between;
- how each declared bound — reaction allowance, freshness, resumption,
  restoration, answer age, the registered cutoff interval — was chosen, and
  what the declaration buys and what it exposes;
- what holding the coverage requirement costs while knowledge comes and goes
  — measured, not asserted.

The document must compare at least two viable designs without turning a known
product's name into the argument, and state one residual limitation.

## Acceptance evidence

No transmission in the recorded history is a violation under the liability
rule — justification recomputed from the coordinator's log, the manifest's
delivery timeline, and the declared allowance, never from the service's
account — and every transmission inside a declared or registered window is
named as exposure with the window that admitted it. Every reported
transmission is honored exactly once, by identity; every authorization is
held by one transmitter and stated with its justifying record; after every
restart and reconnection the service's account agrees with the coordinator's
log. The configured coverage holds over the transmitters the rule assigned
while the service could know and could act, the air and the coverage return
within the declared bounds after every return point, and answers keep coming
throughout — outside the stretches in which the schedule holds every learner
process stopped — each agreeing with the log at the record it claims. Refused
actions and lapses are returned or queryable, never only logged, and the
log's own refusal record confirms the service's.

The report includes the exact authorization, transmission, and answer
histories; every dark stretch with its grounds and its length in records;
coverage over the run and per stretch; the resumption and restoration
distances after each lapse and their worst cases; the exposure account,
transmission by transmission; every refused action and what followed it; the
offered stream rate against the absorbed rate through the bursts; the
declared service level against the calibration reference, with every looser
declaration defended; and latency against the declared service level. It
names what the design gives up to hold both directions of the requirement,
and one residual limitation.

## What a practitioner might have used instead

This lab names none. Every genuine neighbour carries the answer in its
name or on its landing page, so naming one would solve the lab instead
of orienting you. They are in `hints/`.

Stuck? See `hints/`.
