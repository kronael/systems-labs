# Permissionless application hosting

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`HINTS.md`, because saying it here would point straight at the answer.

- **GitHub Pages** — [documentation](https://docs.github.com/en/pages)
- **Arweave** — [documentation](https://docs.arweave.org/)
- **ENS with `contenthash`** — [documentation](https://docs.ens.domains/)

Stuck? See `HINTS.md`.
