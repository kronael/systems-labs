---
status: draft
---

# Multi-chain deposit service

## Brief

Design and build a deposit service of the consumer kind. People deposit funds,
read their own balance and what stands behind it, and withdraw. The service
keeps its records on a local Solana chain and holds the deposited capital at
addresses on two other chains.

The service does not hold the keys to those addresses. To move capital it
composes a transaction and asks a fixed signing network for a signature over
it; the network returns the signature and nothing else, ever. There is no
reply, no receipt, and no way to take a signature back once it has been handed
out. The addresses themselves are ordinary addresses on those chains, so what
the service later sees there is an ordinary transfer, carrying nothing that
says which of the service's own requests produced it, or whether any request
produced it at all. Capital can arrive at those addresses from strangers, and
capital the service authorized can arrive months late or never.

The service can therefore watch the money and cannot ask anyone what it means.
Its records are the only account of who owns what, and they must stay an
account it can defend.

The setting is the custodial deposit arrangement, in which the party holding
the customer relationship is not the party holding the money. United States
banking regulators wrote the invariant down after one such arrangement failed:
records must identify each beneficial owner and the balance attributable to
that owner, they must be reconciled no less often than at the close of
business each day, and the party answerable to the customer must keep
unrestricted access to those records even when the party in the middle has
stopped responding. That is exactly this service's position, and it is why the
numbers below are the size they are. No finance or banking knowledge enters
the lab beyond that paragraph: amounts are integers, and there are no fees, no
interest, no yield, no tax, and no filing to produce.

The public surface is the deposit and withdrawal interface, the per-depositor
account of holdings, and a reader-facing portfolio view over that account. The
concurrency model, the data layout, how capital abroad is attributed and
brought back into agreement with the records, the process decomposition, and
the coordination between the service's components are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts `solana-test-validator` and two further
local chains that hold the deposit addresses. It starts the course-supplied
signing network as a fixed dependency: it accepts a request to sign a composed
transaction, returns a signature after a configurable delay, and reports
nothing afterward — no confirmation, no failure, no history. It includes the
seeded depositor and activity generator; a third-party traffic generator that
sends unrelated transfers into the same addresses; an inspection tool that
reads each chain independently of the service; a fault proxy in front of every
chain and in front of the signing network; and deterministic barriers named
after depositor and authorization identities. Fault scenarios are declarative
files driven by the course-wide fault controller.

The learner owns the service, its on-chain records, its public interface, the
portfolio view, the application Compose layer, and `ARCHITECTURE.md`. The
supported starters are Rust for the chain-facing work and TypeScript for the
view. No cloud account and no mainnet funds are required.

## Requirements

Every depositor has a stable identity, and the service can state that
depositor's balance whenever it is accepting requests. A stated balance is one
the service can honour, and a withdrawal the records say is owed must succeed.

A signature the service requests is an authorization it cannot recall and
cannot follow. Once the network has returned one, the composed transaction may
be included at any later time, or never, and nothing will report which. The
service must decide what a depositor's balance says about capital in that
state, and must hold that decision through every outcome the state can have.

The two deposit chains can be read, but what they show does not name the
service's requests. Movement at an address is attributable to one of the
service's own authorizations only by an argument the service makes; the same
movement may equally be a stranger's transfer, a duplicate inclusion of a
payload the service signed once, or an authorization the service has already
counted. The service must state which of its authorizations it believes each
observed movement discharges, must be able to say why, and must never let an
unattributed movement silently become a depositor's balance.

At the close of each accounting day the service reconciles its records against
the three chains as an independent reader observes them, and publishes the
result. A disagreement is named down to the depositors and authorizations it
involves; it is never absorbed into a balance. The close must complete and
must publish when the signing network has been unreachable for its whole
duration, because the party answerable to the depositor cannot make that
answer conditional on the party in the middle responding.

The portfolio view is a read surface over the public interface and holds no
state of its own. For each depositor it shows the balance, its division across
the chains, and the part resting on an authorization the service has not
attributed. It must never show a total the service would refuse to honour, and
it must keep answering while capital is moving.

The environment fixes two facts. A signature the network returns is
one-directional: it authorizes an outbound transaction and carries no path
back. A signed payload composed so that more than one of the chains will
accept it can be included on a chain it was not intended for.

The scale target is 50,000 depositors and 200,000 deposit and withdrawal
events offered open-loop with at least 128 in flight, capital held under at
least 5,000 distinct authorizations across the two deposit chains with a
declared skew toward one of them, third-party traffic into the same addresses
at four times the service's own authorization rate, and an accounting close
over the full history that finishes within the time the three chains together
take to produce one accounting day of blocks at their own rate. These numbers
size the problem so that a design which attributes movements by matching
amounts, or which stops accepting deposits while an authorization is
outstanding, fails on its own terms. They are not pass thresholds; thresholds
stay relative, calibrated, structural, or learner-declared.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what a depositor's balance is a claim on at each moment, and what makes the
  claim true;
- what the service knows about capital under an unattributed authorization,
  what it does not know, and which of the two the stated balance depends on;
- what evidence attributes an observed movement to one of the service's own
  authorizations, and what that evidence cannot rule out;
- how the service composes an authorization so that it can later recognise
  what that authorization did, and what recognition costs;
- what must be durable before the service asks for a signature, and what
  recovery reads first when it restarts not knowing whether it asked;
- why the close must not depend on the signing network, and what it can still
  assert when that network is silent;
- what a depositor is entitled to read while their own capital is
  unattributed, and why that answer is safe to give;
- how the service keeps accepting deposits while earlier authorizations remain
  unattributed, and what bounds the exposure that creates;
- which properties the local chains and the local signing network cannot prove
  about their public counterparts.

The document must compare at least two designs for what a stated balance
counts, and must say what each costs the depositor when an unattributed
authorization resolves the other way.

`HINTS.md`-bound, because each presupposes a mechanism: how a composed
transaction is bound to a single destination chain so it cannot be accepted
elsewhere; how an outbound transfer is made self-identifying to its own
sender without a reply channel.

## Adversarial evaluation

The failure schedule fires at named barriers, never on a timer and never at
random:

- it stops the signing network at the barrier where a named depositor's
  withdrawal is accepted, and holds it stopped through the next accounting
  close, so the close must publish with the middle party silent;
- it kills the service between the request for a named depositor's
  authorization and the recording that the request was made, then restarts it
  against the same chains, so the service returns not knowing whether capital
  was authorized to leave;
- it delivers a named authorization's signed payload to the second deposit
  chain as well as the intended one, and includes it on both;
- it includes a named authorization's transaction after the service has
  written that authorization off, and at the same barrier sends a stranger's
  transfer of the same amount to the same address;
- it accepts a named depositor's full withdrawal at the moment that
  depositor's capital rests on an authorization the service has not
  attributed;
- it releases a named authorization's signature only after the accounting
  close that expected it has already published.

Evaluation observes the three chains through their own interfaces, the
service's public interface and portfolio view, process lifecycle, and exact
per-depositor and per-authorization histories. It does not require a named
recovery pattern, and it keeps watching after each close — a balance that
becomes unhonourable after it was stated is a failure regardless of counts.

## Acceptance evidence

Every depositor's stated balance is honourable at every point the service
accepted requests, including through each named barrier. No observed movement
is attributed to two authorizations, and no stranger's transfer becomes a
depositor's balance. Every close either publishes agreement or names the
depositors and authorizations it could not agree on, and the close that runs
with the signing network stopped still publishes. The doubly-included payload
is attributed once. The written-off authorization that later lands is
recovered without paying it out twice and without confusing it with the
stranger's transfer at the same barrier.

The report records, for every authorization: the depositor, the destination
chain, the moment of the signature request, the movement the service
attributed to it with the evidence for that attribution, and the state the
service assigned it in between. For every close it records duration against
the chains' own block rate, the count and identity of unagreed items, and how
long each stayed unagreed. It records the portfolio view's answers alongside
the independently read chain state at the same instants. It closes by
contrasting, with the observed histories as evidence, what the service could
assert about capital abroad before and after each close, and what a depositor
would have been told at each of those points.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **NEAR Intents** —
  [documentation](https://docs.near-intents.org/near-intents). Settles a
  user's stated intent through competing solvers who take the other side, so
  capital passes between parties rather than resting at an address someone
  must later account for, and no operator has to decide what an unexplained
  movement means.
- **Wormhole Portal** —
  [documentation](https://wormhole.com/docs/protocol/introduction/). Locks the
  original asset and mints a wrapped claim against it, so the wrapper's own
  supply is the record of who owns what and attribution against an independent
  reading of the far chain is not a question the design has to answer.
- **THORChain** —
  [documentation](https://docs.thorchain.org/how-it-works/technology). Holds
  capital in a shared vault secured by bonded operators and slashing, so
  solvency is defended by making misattribution expensive for the operator
  rather than by an account the operator must be able to state and prove.

Read their documentation on custody, settlement, and how each answers for
capital in transit. The lab does not run them.

## Scope

The expected focused time is eighteen to twenty-four hours. The three chains,
the signing network, the depositor and third-party generators, the fault
schedules, and the independent chain reader are prepared. Deposits and
withdrawals use each chain's native transfer; yield, pricing, fees, interest,
token standards, exchange between assets, governance, and the visual design of
the portfolio view are outside the problem. Chain reorganization is outside
this lab and belongs to `7/3`; retry identity on a chain the sender can query
belongs to `7/4`. The service earns nothing and loses nothing; the only thing
it must get right is who owns what.

No step requires mainnet funds or a cloud account. Public RPC endpoints are
opt-in, bounded, cached under the shared source directory, and never on a
request path. CI and checks use the local chains and the local signing network
only.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, fault,
  evidence, and verification contracts.
- [`../0/3-blockchain-track.md`](../0/3-blockchain-track.md) — track record
  and the origination of this candidate.
- [`3-finality-aware-transfer-index.md`](3-finality-aware-transfer-index.md)
  — the neighbouring lab whose subject is what a receipt proves when the chain
  can withdraw it. There the movement names itself and the problem is that the
  chain takes it back. Here the movement never names itself and the chain
  never takes anything back. Reorganization belongs to that lab and is out of
  scope here.
- [`4-reliable-transaction-dispatcher.md`](4-reliable-transaction-dispatcher.md)
  — the neighbouring lab whose subject is when a resubmission is the same
  payment. It holds the keys and can ask the chain what happened. This lab
  holds no key abroad and has nobody to ask.
- [NEAR chain signatures](https://docs.near.org/chain-abstraction/chain-signatures)
  — the signing network's real counterpart, and the source of this lab's
  central constraint: it is "a 'one way' solution to sign and execute outbound
  transactions happening on other blockchains", addresses are derived
  deterministically so they are ordinary addresses on the destination chain,
  and its documentation warns against signatures that can be "replayed on a
  chain you did not intend to interact with".
- [FDIC, Recordkeeping for Custodial Accounts, 89 FR 80135](https://www.govinfo.gov/content/pkg/FR-2024-10-02/html/2024-22565.htm)
  — the domain fact that makes the invariant and the daily close inevitable.
  Proposed 12 CFR 375.3(b) requires "Maintaining accurate balances of
  custodial deposit accounts with transactional features at the beneficial
  ownership level" and "Conducting reconciliations against the beneficial
  ownership records no less frequently than at the close of business daily";
  375.3(c)(1) requires "direct, continuous, and unrestricted access to the
  records … including in the event of business interruption, insolvency, or
  bankruptcy of the third party". The same document records why: after the
  Synapse bankruptcy, institutions "encountered significant difficulties in
  obtaining, reviewing, and reconciling Synapse's records", and "the deposits
  at the IDIs appear to be insufficient to cover the amounts owed by the
  fintech companies to their customers".
- Implementation pointers do not exist while the spec is `draft`.
