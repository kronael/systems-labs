> Spoilers. Open only when stuck.

# Spectrum compliance service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule fires at named records and acknowledgements, never on a
timer, and `make fault` writes what it fired into the run manifest under
`evidence/`: the suppressed, proving, lifting, healing, and return records,
the named transmitters, authorizations, and transmissions, and a transcript
of every answer it read with the newest record the coordinator had emitted at
that moment. Checks select on the manifest and the coordinator's log, never
on literals, and never on the service's account.

It suppresses the record after a named stream record and delivers the ones
that follow, leaving the session healthy; when delivery reaches the first
record past the proof record plus the declared reaction allowance, it drives
traffic at a named transmitter if that transmitter still stands on the air on
its pre-gap justification. A transmission is a violation; transmissions
landing before the allowance's edge are exposure the evidence must name. What
must hold after: from the proof record, the first transmitter on the air
within the resumption bound and coverage within the restoration bound, from
an account that agrees with the log.

It repeats the suppression as a run of single-record gaps at named records,
spaced closer together than the whole fleet could be re-authorized under the
action ceiling. Coverage across the stretch is judged at the configured
fraction: the design pays for what each gap actually cost it, and a design
that buys every gap by tearing its authorizations down and asking again — or
by taking the whole fleet dark — spends the stretch below the fraction and
fails here.

It severs the session at the exact moment a named authorization's renewal is
acknowledged — the acknowledgement that named its expiry — leaves the stream
flowing, and holds the severance past that expiry, lifting it at a named
record after the lapse is on the public record. The checks here are orderings
in the log, never record literals: the close, then the lapse exactly at the
expiry the acknowledgement named, and no transmission of the named
transmitter after the expiry — transmissions up to it ride the authorization
the acknowledgement already bought, and are justified. When delivery reaches
the first record past the lapse, the schedule drives traffic at the named
transmitter; a transmission is the violation the enforced expiry names. What
must hold after the lift: a session re-established, the lapsed authorization
renewed or re-acquired, the named transmitter on the air within the
resumption bound, coverage within the restoration bound — a fleet's worth of
renewals through a 500-action ceiling, paced, not panicked.

In the same instant it delivers a named record whose consequence under the
supplied rule moves a named transmitter's assignment, it stops every learner
process, so nothing that record provokes can reach the coordinator or the
fleet; while the pause holds, the stream moves on, a further named record
moves that assignment again, and the session closes. While the pause holds,
the schedule drives traffic at the named transmitter: a transmission inside a
registered cutoff's countdown is exposure that arrangement bought, named in
the evidence, and the cutoff must fire when due — the leaving of the air,
then no transmission after the due point, orderings in the monitor's log. A
design that registered no cutoff leaves the transmitter on the air, and the
schedule drives a transmission one largest-acceptable interval after the
service's last contact with it — the violation that design bought. At a named
record after the close it resumes the processes. The late action must not
take effect — the closed session refuses what was bound for the coordinator,
and an instruction landing on a transmitter after the pause whose justifying
record predates the pause is a violation the moved assignment makes visible
in the monitor's log. While every learner process is held stopped, and only
then, the answers requirement is suspended; the first answer after resumption
states truthfully what it is true of.

It cuts the stream's delivery outright at a named record and refuses every
rejoin until it heals delivery at a named record, holding the cut longer, in
emitted records, than the declared answer age; the session stays live
throughout, and renewals sent on it keep succeeding. Answers must keep coming
through the cut, aging honestly, until the declared age refuses them with the
grounds stated. When the silence has outlasted the cadence and the declared
allowance, it drives traffic at a named transmitter if that transmitter still
stands on the air — a violation if a transmission lands, however current its
renewals: an authorization kept alive proves nothing about the state it lives
in. From the heal, the return is checked within the declared bounds.

When a named transmission is appended to the coordinator's log, it kills
every learner process outright, suspends a named transmitter's authorization
while they are down, then restarts them at a named record. Afterwards the
account is rebuilt to agreement with the log: every transmission reported
while the service was down is honored exactly once by identity, every
authorization is mapped to its one transmitter, no transmitter returns to the
air except on a record the rebuilt account can state — and the suspended
transmitter stays dark until its reinstatement is delivered, however the
fleet last stood.

Inside every one of these stretches the workload moves at least one
assignment the schedule later asks about, so an answer claiming sight of it
is checkable against the log; throughout every stretch the schedule reads the
service's account and requires answers that state truthfully what they are
true of.

At the end the coordinator's log, the manifest's delivery timeline, and the
service's account are read together. Justification is recomputed for every
transmission from the log, the timeline, and the declared allowance: no
transmission is a violation under the liability rule, every transmission
inside a declared or registered window is named as exposure, every reported
transmission is honored exactly once by identity, every authorization was
held by one transmitter, every answer in the transcript agrees with the log
at the record it claims, and the air and the coverage returned within the
declared bounds after every return point.

Checks observe only the coordinator's log and stream, the service's public
interface, process lifecycle, metrics, the run manifest, and the submitted
evidence. They do not require a particular internal structure, persistence
choice, or coordination primitive.
