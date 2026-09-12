---
status: draft
---

# Permissionless application hosting

## Brief

Design and build a system that publishes an application — an on-chain program
and the interface that drives it — and delivers it to users with no server the
publisher operates, no domain the publisher rents, and no account anywhere. A
user who knows one published name obtains the current interface release,
verifies that its bytes are exactly what the publisher released, and uses the
program. The application keeps working after the publisher's machine is off.

The application itself is deliberately small — a public notice board: any user
appends a short signed entry and reads the board — because the product under
study is the delivery, not the program. The system covers the whole life of
that delivery: publishing a release, keeping every release retrievable, moving
users to the newest one, and ending with a program that nobody, including the
publisher, can change.

Deployment buys less than it appears to. Nothing keeps
the published bytes available unless something is arranged to keep them; the
address of the bytes can never change, so being current lives in a separate
pointer that can go stale or die of neglect; and the program remains
replaceable underneath every user until the right to replace it is given up,
which can be done exactly once and never undone. Each of these has to be
established deliberately and proved, not assumed.

The assignment is the whole system: the program, the interface releases, the
publishing path, and end-to-end tests. The replication and pinning strategy,
the pointer design, the release process, the key-custody scheme, and the
client architecture are the learner's decisions.

## Prepared scaffold

The supplied Compose profile extends the standard scaffold with a local
`solana-test-validator` whose ledger persists across restarts, funded with
airdropped lamports, and with a name registry program deployed at genesis. A
registry record binds a name to a small byte value under the record convention
the public Solana name service uses for content identifiers. The run's
published name is fixed by the lab config.

The profile also starts a private content network of four Kubo (IPFS) nodes:
one publishing node the submission attaches to and may configure; two standing
peers that stay up for the whole run and accept pin requests over the node
API, standing in for the paid pinning services of the public network; and one
user-side node the submission cannot configure, through which every user agent
resolves and fetches.

The environment's facts are current as of 2026-08-14. On the content network:
a block no pin protects is removed when a node collects garbage, and the
network guarantees that content is discoverable, not that it stays available.
In the node's naming layer, a name is the hash of a key; its record carries a
validity; copies placed in the DHT expire after 48 hours regardless of that
validity; the publishing node republishes every 4 hours by default, and only
while it runs; and resolving a name costs round trips that resolving a content
address does not. On the chain: a deployed program carries an upgrade
authority, by default the deploying wallet, that can replace or close it;
removing the authority makes the program immutable; an immutable program can
never be updated or closed; and a closed program's address can never host a
program again.

The course supplies the release plan and payload generator, which emits each
release's seeded asset payloads at the sizes and deltas the scale target
names; the user agent fleet, which drives any submission through a fixed
manifest contract included in the starter; and the fault controller with
declarative scenario files.

The learner owns the program, in Rust; the interface bundle and its manifest,
in TypeScript; the publishing path; and `ARCHITECTURE.md`. No cloud account,
no registrar, no pinning-service subscription, and no mainnet funds are
involved. The public content network, public pinning services, and public RPC
endpoints are opt-in, bounded, cached, and never on a request path; no gate
requires them.

## Requirements

A user agent that knows only the published name obtains the current release,
verifies its bytes, and drives the program through it. Every release has a
stable identity, and the bytes under an identity never change: a release
resolves to exactly the bytes published for it, a superseded release remains
reachable by its own address for the whole run, and no release ever resolves
to bytes published under a different one.

Publishing moves users forward. After a release is published, a fresh user
obtains it; how long that takes is measured. When the system cannot make or
know a release current, the user gets an explicitly labeled older release or
an explicit failure — never an unlabeled stale answer and never bytes that mix
releases.

The current release remains obtainable while the publishing node is stopped.
What makes that true is the design's own business; naming the party it trusts
is required evidence.

The system answers, at every moment and truthfully against chain state,
whether the program can still change and who can change it. The release plan
contains one release that changes the program. A user driving the application
through a superseded interface after that change receives an explicit refusal
or an answer explicitly marked degraded; a silent wrong effect on the board is
a failure. The terminal release leaves the program immutable, and the report
states what immutability forecloses.

The scale target is 24 releases published on the course release plan; every
interface bundle at least 48 MiB with at most 8 MiB of new bytes between
consecutive releases, so the run addresses about 1.1 GiB of content of which
about 230 MiB is distinct; ten mid-run releases published one minute apart; 16
user agents fetching concurrently, each starting cold; and 25 board operations
per second sustained through fetched interfaces. These numbers size the
problem: the cadence outruns comfortable pointer propagation so staleness is
observable, the bundle size makes a cold fetch a measured cost rather than a
rounding error, and the history makes keeping everything everywhere a real
cost. They are not pass thresholds; thresholds stay relative, calibrated,
structural, or learner-declared, per the verification contract.

The evidence must show cold and warm retrieval time against the learner's
declared service level, publish-to-fresh-user delay per release, observed
pointer staleness, distinct bytes against addressed bytes across the history,
and lamports spent on deployment, registry records, and fees. This lab
requires those measurements because the delivery is the lesson; how they are
collected is the learner's choice.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what holds a release's bytes available after the publishing machine is
  gone, which party that arrangement trusts, and how its cost grows with the
  release history;
- where the pointer that makes a release current lives, who can move it, what
  keeps it alive when nobody moves it, and how long a user can still observe
  the old answer after it moves;
- how a user distinguishes current, stale, and gone, and which of the three
  the design can detect from the user's side of the network;
- which keys exist, what each one can do, and what the loss and the theft of
  each one costs;
- when the upgrade authority should be revoked, what revocation forecloses,
  and how the interface keeps evolving against a program that can no longer
  change;
- how the encounter between a superseded interface and a changed program is
  made loud rather than silently wrong;
- what reachability of the application means, measurably, once the publisher
  is off, and the one residual dependency the design stands on.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

Faults fire at named barriers, never on a timer and never at random. The
schedule includes:

- at the barrier where release 17's pointer update is acknowledged, the
  publishing node is stopped and stays stopped; a cold agent then asks for the
  current release;
- at a named barrier, the pins protecting release 9 on one named standing
  peer are removed and that peer collects garbage; a cold agent then fetches
  release 9 by its own address;
- the pointer update for release 21 is withheld at the boundary until a named
  barrier passes, so the pointer stays on release 20 while release 21 exists;
  agents keep resolving throughout the window;
- the pointer maintenance path is suspended and the clock is advanced past
  the pointer's own validity; agents keep resolving;
- release 12 changes the program; an agent that fetched release 11's
  interface before the change continues to drive the program after it;
- after the terminal release, the harness attempts a program upgrade with the
  same credentials the run used, and then uses the application.

Verification observes only the user-side node and the validator's RPC. It
records, for every agent and every fetch, the release identity presented and
a checksum of the bytes delivered, and it compares the board's final state
against an independently computed one. It does not inspect the publishing
path and does not require a named replication or pointer mechanism.

## Acceptance evidence

Every one of the 24 releases is retrieved byte-exact by a cold agent at the
end of the run, by its own address. No fetch in the entire history delivered
bytes that differ from the release identity presented with them. Release 17
is retrieved during the publisher outage. Release 9 is retrieved after the
named peer collected garbage. During the withheld-pointer window every answer
is release 20 identified as release 20, and the system's own status shows the
staleness. During the expired-pointer window every resolution is an explicit
failure or an explicitly labeled last-known release; no agent receives an
unlabeled answer. The agent holding release 11's interface after the program
change receives an explicit refusal or a marked-degraded answer, and the
board matches the independent computation. The post-final upgrade attempt
fails on chain, and the application still serves.

The evidence report includes cold and warm retrieval time against the
declared level, publish-to-fresh-user delay for every release including the
one-minute cadence window, the distribution of observed pointer staleness,
the content each environment node holds at the end of the run against the
distinct bytes of the history, and lamports spent on deployment, registry
records, and fees. It closes with the availability statement: one paragraph
naming what keeps the application reachable while the publisher's machine is
off and the residual dependency that arrangement stands on. Every design has
one; the report is graded on naming it, not on pretending it is absent.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **GitHub Pages** — [documentation](https://docs.github.com/en/pages). Static
  hosting behind a CDN makes availability an operator's obligation instead of
  the publisher's arrangement: the site stays up because the host keeps it up,
  the name moves the instant the operator says so, and the price is that
  nothing binds the name to particular bytes and the account is the
  availability — published sites may be no larger than 1 GB, bandwidth has a
  soft limit of 100 GB per month, and the host's terms decide what stays
  published.
- **Arweave** — [documentation](https://docs.arweave.org/). Makes permanence a
  purchase rather than an arrangement: one upfront fee, most of which enters a
  storage endowment that pays out only when block rewards fall short, priced
  on the expectation that storage keeps getting cheaper. Nothing needs the
  publisher after the transaction and nothing can be unpublished — and the
  mutable-pointer problem remains exactly as it is here.
- **ENS with `contenthash`** — [documentation](https://docs.ens.domains/).
  Puts the pointer itself on a chain: a resolver record holds a
  machine-readable multicodec content address, formerly standardized as
  EIP-1577, so moving the pointer is a transaction, the record persists with
  no republishing publisher, and every past value is public — at the cost of
  gas per move and of needing a chain client or a trusted gateway to read it.

Read the GitHub Pages usage limits, the Arweave endowment description, and
the ENS contenthash specification. The lab does not run them.

## Scope

The expected focused time is twenty to twenty-five hours. The learner builds
the program, the interface bundle and manifest, and the publishing path. The
validator, the name registry, the four-node content network, the user agent
fleet, the release plan and payload generator, and the fault schedules are
prepared.

Every required gate runs locally at zero cost and with no account of any
kind: no cloud account, no domain registrar, no pinning-service subscription,
and no wallet holding real funds. The lab's subject is removing the accounts,
so it must not require one. Optional use of the public content network, a
public pinning service, or public RPC is opt-in, bounded, cached under the
shared source directory, and never on a request path; CI and verification
use the local environment only.

Program runtime limits belong to the settlement lab, transaction dispatch
reliability to the dispatcher lab. DNS and real domains, TLS and public
gateways, token standards, multi-validator clusters, and browser-specific
packaging are outside the problem.

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold, fault,
  evidence, and verification contracts.
- [`../../docs/blockchain-track.md`](../../docs/blockchain-track.md) — track record
  and the origination of this candidate.
- [IPFS persistence and pinning](https://docs.ipfs.tech/concepts/persistence/)
  — the network guarantees content is discoverable, not persistently
  available; data persists only while pinned, and pinning services run nodes
  that pin data for a fee.
- [Pinning and garbage collection in practice](https://docs.ipfs.tech/how-to/pin-files/)
  — direct, recursive, and indirect pins; collecting garbage removes every
  unpinned object from the node.
- [IPNS](https://docs.ipfs.tech/concepts/ipns/) — a name is the hash of a
  key; DHT copies of a record expire after 48 hours regardless of the
  record's validity; Kubo republishes every 4 hours by default while it runs;
  resolution is slower than a content address because multiple records must
  be found. Solution-bearing for the pointer design: this belongs in
  `HINTS.md`, never in `README.md`.
- [Deploying programs](https://solana.com/docs/programs/deploying) — the
  upgrade authority defaults to the deploying wallet and can update or close
  the program; `--final` removes it; once a program is immutable it can never
  be updated or closed, and a closed program's address can never be reused.
- [Solana name service records](https://guide.sns.id/domain-name/records.html)
  — the record convention the prepared registry follows: web3 record types
  including an IPFS content identifier and an Arweave address bound to a
  name. Solution-bearing for the pointer design: this belongs in `HINTS.md`,
  never in `README.md`.
- [ENSIP-7 contenthash](https://docs.ens.domains/ensip/7/) — the on-chain
  pointer as practiced on Ethereum: a multicodec content address in a
  resolver record, formerly EIP-1577.
- [Arweave storage endowment](https://www.arweave.com/blog/endowment-with-arweave)
  — pay once, store forever: most of the fee enters an endowment released
  only when block rewards cannot cover storage, on the assumption of
  declining storage costs.
- [GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)
  — the hosted neighbour's terms: 1 GB site, 100 GB per month soft bandwidth
  limit, ten builds per hour.
- Implementation pointers do not exist while the spec is `draft`.
