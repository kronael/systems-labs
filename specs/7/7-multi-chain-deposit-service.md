---
status: draft
---

# Multi-chain deposit service

## Brief

Design and build a deposit service of the consumer kind. People deposit funds,
read their own balance and what stands behind it, and withdraw. The service
keeps its records on a local Solana chain and puts the deposited capital to
work on two other chains, so at any moment a depositor's money is partly on
the chain that records it and partly somewhere the record cannot see.

The service holds no key on those other chains. To move capital there it asks
a fixed signing network for a signature over a transaction it has composed;
the network returns the signature and nothing else, ever. There is no reply
path, no receipt, and no way to take a signature back once it has been handed
out. The service's own records are therefore the only account of who owns
what, and they must stay an account it can defend.

The setting is the custodial deposit arrangement, in which the party holding
the customer relationship is not the party holding the money. United States
banking regulators wrote the invariant down after one such arrangement failed:
records must identify each beneficial owner and the balance attributable to
that owner, they must be reconciled against the holder's records no less often
than at the close of business each day, and the party answerable to the
customer must keep unrestricted access to them even when the party in the
middle has stopped responding. That is exactly this service's position, and it
is why the numbers below are the size they are. No finance or banking
knowledge enters the lab beyond that one paragraph: amounts are integers,
there are no fees, no interest, no tax, and no regulatory filing to produce.

The public surface is the deposit and withdrawal interface, the per-depositor
account of holdings, and a reader-facing portfolio view over that account. The
concurrency model, the data layout, how capital abroad is tracked and brought
back into agreement with the records, the process decomposition, and the
coordination between the service's components are the learner's decisions.

## Prepared scaffold

The supplied Compose stack starts `solana-test-validator`, a local Ethereum
development node configured for block-interval mining, and a third local chain
with a slower and independently configurable inclusion rule, so the three do
not agree about when capital has moved. It starts the course-supplied signing
network as a fixed dependency: it accepts a request to sign a composed
transaction for a named destination chain, returns a signature after a
configurable delay, and reports nothing afterward. It includes the seeded
depositor and activity generator, an inspection tool that reads each chain
independently of the service, a fault proxy in front of every node and in
front of the signing network, and deterministic barriers named after depositor
and authorization identities. Fault scenarios are declarative files driven by
the course-wide fault controller.

The learner owns the service, its on-chain records, its public interface, the
portfolio view, the application Compose layer, and `ARCHITECTURE.md`. The
supported starters are Rust for the chain-facing work and TypeScript for the
view. No cloud account and no mainnet funds are required.

## Requirements

Every depositor has a stable identity, and the service can state that
depositor's balance at any moment it is accepting requests. Every stated
balance is one the service can honour: a withdrawal the records say is owed
must succeed, and the service must never state a balance that two depositors
could both draw against. Capital the service holds must always be attributable
to some depositor, and no capital may be attributable to two.

A signature the service requests is an authorization it cannot recall. Once
the signing network has returned one, the composed transaction may be included
on its destination chain at any later time, or never, and the service will not
be told which. The service must classify capital in that state, must say what
its classification means to a depositor reading a balance, and must not let
either possible outcome break the attribution requirement above.

At the close of each accounting day the service reconciles its records against
the three chains as an independent reader observes them. A disagreement must
be reported through the public surface, named down to the depositors and
authorizations it involves, and must never be silently absorbed into a
balance. The reconciliation must still complete and still report when the
signing network is unreachable for the whole close, because the party
answerable to the depositor cannot make that answer conditional on the party
in the middle responding.

The portfolio view is a read surface over the public interface and holds no
state of its own. For each depositor it shows the balance, its division across
the three chains, and the part that is under an authorization whose outcome is
not yet observed. It must never show a total that the service would refuse to
honour, and it must keep answering while capital is moving.

The environment fixes three facts. A signature the network returns is
one-directional: it authorizes an outbound transaction and carries no path
back. A composed Solana transaction is valid only while the blockhash it
carries stays within the node's most recent 151, about 60 to 90 seconds, so an
authorization can expire between signing and inclusion. The same signed
payload, if it is composed so that more than one chain will accept it, can be
included on a chain it was not intended for.

The scale target is 50,000 depositors and 200,000 deposit and withdrawal
events offered open-loop with at least 128 in flight, capital deployed abroad
under at least 5,000 distinct authorizations across the two foreign chains
with a declared skew toward one destination, and an accounting close over the
full history that finishes within the time the three nodes together take to
produce one accounting day of blocks at their own rate. These numbers size the
problem so that a design which reconciles by rereading everything, or which
stops accepting deposits while capital is in flight, fails on its own terms.
They are not pass thresholds; thresholds stay relative, calibrated,
structural, or learner-declared.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what a depositor's balance is a claim on at each moment, and what makes the
  claim true;
- what the service knows about capital under an unobserved authorization,
  what it does not know, and which of the two the stated balance depends on;
- why the attribution requirement cannot be satisfied by counting the same
  capital in the way that is most convenient for each caller;
- what must be durable before the service asks for a signature, and what
  recovery reads first when it restarts not knowing whether it asked;
- how the close establishes agreement between the records and three chains
  that became consistent at different times, and what it does with the part
  that has not;
- why the close must not depend on the signing network, and what it can still
  assert when that network is silent;
- what a depositor is entitled to read while their own capital is in motion,
  and why that answer is safe to give;
- how the service keeps accepting deposits while an earlier authorization is
  unresolved, and what bounds the exposure that creates;
- which properties the local chains and the local signing network cannot
  prove about their public counterparts.

The document must compare at least two designs for what a stated balance
counts, and must say what each design costs the depositor when the unobserved
case resolves the other way.

`HINTS.md`-bound, because each presupposes a mechanism: how a composed
transaction is bound to one destination chain so it cannot be accepted
elsewhere; how an expiring validity window is removed from a transaction that
must be signed before it can be sent.

## Adversarial evaluation

The failure schedule fires at named barriers, never on a timer and never at
random:

- it holds a named authorization inside the signing network until after the
  blockhash of the transaction it signs has expired, then releases the
  signature, so the service holds an authorization that can never be included
  and will never be refused;
- it kills the service between the request for a named depositor's
  authorization and the recording that the request was made, then restarts it
  against the same chains, so the service comes back not knowing whether
  capital was authorized to leave;
- it delivers a named authorization's signed payload to the second foreign
  chain as well as the intended one, and includes it on both;
- it stops the signing network entirely at the barrier where a named
  depositor's withdrawal is accepted, and holds it stopped through the next
  accounting close;
- it accepts a named depositor's full withdrawal at the moment that
  depositor's capital is under an authorization the service has not observed;
- it rolls back the foreign chain block carrying a named authorization after
  the service has observed that authorization included.

Evaluation observes the three chains through their own interfaces, the
service's public interface and portfolio view, process lifecycle, and exact
per-depositor and per-authorization histories. It does not require a named
recovery pattern, and it keeps watching after each close — a balance that
becomes unhonourable after it was stated is a failure regardless of counts.

## Acceptance evidence

Every depositor's stated balance is honourable at every point the service
accepted requests, including through each named barrier. No capital is
attributed to two depositors and none to none. Every close either reports
agreement or names the depositors and authorizations it could not agree on,
and the close that runs with the signing network stopped still produces that
report. The withdrawal accepted against in-flight capital resolves without
paying out capital that never landed and without stranding capital that did.
The doubly-included authorization is attributed once.

The report records, for every authorization: the depositor, the destination
chain, the moment of the signature request, the moment of observed inclusion
where one exists, and the classification the service gave it in between. For
every close it records duration against the nodes' own block rate, the count
and identity of unagreed items, and how long each stayed unagreed. It records
the portfolio view's answers alongside the independently read chain state at
the same instants. It closes by contrasting, with the observed histories as
evidence, what the service could assert about capital abroad before and after
each close, and what a depositor would have been told at each of those points.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **NEAR Intents** —
  [documentation](https://docs.near-intents.org/near-intents). Settles a
  user's stated intent through competing solvers who take the other side, so
  capital passes between parties rather than resting in a pool, and no
  operator ever has to state a balance for capital whose whereabouts it
  cannot currently observe.
- **Wormhole Portal** —
  [documentation](https://wormhole.com/docs/protocol/introduction/). Locks the
  original asset and mints a wrapped claim against it, so the wrapper's own
  supply is the record of who owns what and reconciliation against an
  independent reading of the far chain is not a question the design has to
  answer.
- **THORChain** —
  [documentation](https://docs.thorchain.org/how-it-works/technology). Holds
  capital in a shared vault secured by bonded operators and slashing, so
  solvency is defended by making misattribution expensive for the operator
  rather than by an accounting rule the operator must be able to state and
  prove.

Read their documentation on custody, settlement, and how each answers for
capital in transit. The lab does not run them.

## Scope

The expected focused time is eighteen to twenty-four hours. The three chains,
the signing network, the depositor and activity generator, the fault
schedules, and the independent chain reader are prepared. Deposits and
withdrawals use each chain's native transfer; yield, pricing, fees, interest,
token standards, exchange between assets, governance, and the visual design of
the portfolio view are outside the problem. The service earns nothing and
loses nothing; the only thing it must get right is who owns what.

No step requires mainnet funds or a cloud account. Public RPC endpoints are
opt-in, bounded, cached under the shared source directory, and never on a
request path. CI and checks use the local chains and the local signing network
only.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, fault,
  evidence, and verification contracts.
- [`../0/3-blockchain-track.md`](../0/3-blockchain-track.md) — track record
  and the origination of this candidate.
- [`4-reliable-transaction-dispatcher.md`](4-reliable-transaction-dispatcher.md)
  — the neighbouring lab in this phase. It drives both chains with keys it
  holds and asks whether each payment landed exactly once. This lab holds no
  key abroad and cannot ask, so its question is what may be published about
  capital whose fate is unknown. The two must not be run as one.
- [NEAR chain signatures](https://docs.near.org/chain-abstraction/chain-signatures)
  — the signing network's real counterpart, and the source of this lab's
  central constraint: it is "a 'one way' solution to sign and execute outbound
  transactions happening on other blockchains", and its documentation warns
  against signatures that can be "replayed on a chain you did not intend to
  interact with".
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
- [Transaction confirmation and expiration](https://solana.com/developers/guides/advanced/confirmation)
  — a transaction is valid while its blockhash sits within the 151 most
  recent, about 60 to 90 seconds, after which it will never be processed.
- [Durable nonces](https://solana.com/docs/core/transactions/durable-nonces)
  — replaces the expiring blockhash with a stored value that a transaction's
  required first instruction advances, removing the validity window.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
