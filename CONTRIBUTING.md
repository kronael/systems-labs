# Contributing

This file is for whoever writes labs. If you came here to take the labs, read
[`README.md`](README.md) instead — and be aware that everything below is
solution-bearing about the curriculum's design.

## Where things live

- [`specs/index.md`](specs/index.md) — the authoritative catalog. Every lab's
  system, architecture pressure, prepared environment, and status.
- [`specs/01-systems-labs.md`](specs/01-systems-labs.md) — the governing spec.
  Every cross-lab contract lives here and nowhere else. Where this file and the
  spec disagree, the spec wins.
- [`specs/<phase>/`](specs/) — the lab specs, one directory per phase, plus a
  `README.md` orienting the phase.
- [`specs/0/5-shared-scaffold.md`](specs/0/5-shared-scaffold.md) — the one
  generator, fault controller, evidence writer, and template all 34 labs share.
- [`docs/`](docs/) — the selection record and the track catalogs: which labs
  were chosen, which were cut, and the candidates behind phases 6, 7 and 8.
- [`HOWTO.md`](HOWTO.md) — the learner's method: the loop, the gates, what
  they write, and when they open which file. The counterpart to this file.
- [`BUGS.md`](BUGS.md) — the review queue: what is wrong with what exists.
- [`TODO.md`](TODO.md) — what does not exist yet, and which proposals are
  still missing the documented behaviour that would let them become specs.
- [`.diary/`](.diary/) — the shipping log, dated. History lives here and is
  never narrated into the specs.

`specs/` is the authoring source. The learner tree is derived from it, and no
learner reads a spec directly.

## The teaching contract

One rule governs the rest: **a learner who wants the exercise must be able to
avoid the answer without effort.**

A brief reads "design and build a system that does X under these requirements."
It fixes the product, public behaviour, environment, limits, failures, and
evidence. It never gives the service decomposition, schema, event keys,
transaction boundaries, retry algorithm, cache policy, recovery mechanism, or
deployment topology.

**A disclaimed mechanism is still a named mechanism.** Writing "does not
prescribe a worker pool" tells the learner a worker pool is in play. A brief
names the *categories* of decision the learner owns and no candidate under any
of them, not even to exclude it.

`README.md` carries the requirements and the technology landscape, and nothing
else. No techniques, no data structures, no invariants of a design's own, no
complexity analysis, and none of the phrases "the trick", "the trap", "the
catch", or "the hard part". Lab titles and directory slugs are solution-neutral
too — names are part of the prompt.

Every `Code pointers` citation is solution-bearing and belongs in `hints/` or
`EVALUATION.md`. The page that reports a behaviour states the behaviour, so
citing it in `README.md` hands over the reading.

`make teaching-lint` enforces this and runs in CI.

## Faults fire at named barriers

A scenario says "when record 4711 is acknowledged, freeze the broker" — never
on a timer, never at random — so the failure lands at the same boundary on
every run. Random chaos proves nothing twice. Every scenario declares what must
remain true after recovery.

## The domain supplies reasons, never difficulty

A lab names a real subject, and that subject explains why the invariant exists
and why the scale target's numbers are the size they are. A domain that could
be swapped for any other without changing a requirement is decoration.

The opposite failure is worse because it is invisible: the domain must never
become something to learn. No formats to parse, no vocabulary to memorize, no
regulation to interpret. One sentence of fact that explains an existing
requirement is the whole of it.

## Adding a lab

1. Find the real reported behaviour first — a manual page, a post-mortem, a
   paper, a vendor limit. Fetch the page and confirm it says what you claim; a
   URL recalled from memory is not a citation. Never invent a quirk and then go
   looking for a source.
2. Write the spec under `specs/<phase>/` with the nine sections in order and no
   others: `Brief` · `Prepared scaffold` · `Requirements` ·
   `Architecture questions` · `Adversarial evaluation` · `Acceptance evidence` ·
   `Neighbouring systems` · `Scope` · `Code pointers`. Frontmatter carries one
   key, `status:`.
3. Name the scale target — speed, load, amount — and state what the naive small
   tool would fail at that scale. If nothing, raise the target or drop the
   dependency.
4. Ground the domain in a cited fact. If the spec reads the same with the
   domain swapped out, ground it or drop the pretence.
5. Split `Architecture questions`: a question stated at the level of a property
   publishes into `README.md`; one that presupposes a mechanism is
   `hints/`-bound.
6. Write `EVALUATION.md`. Never write a worked solution.
7. Add the fault schedule recipe to the controller and update the frozen
   digest.
8. Run `make teaching-lint`, then add a row to the core catalog in
   `specs/index.md`.

## Filing a defect

Record it in [`BUGS.md`](BUGS.md); do not fix it in passing. The queue is how
work gets prioritised, and a silent fix loses the reasoning.

A fix that changes a contract, control flow, or anything cross-cutting is a
redesign: record it as a proposal and get sign-off before shipping it. Closed
findings are pruned to `.diary/` so the queue stays readable.

## Approval boundary

`specs/01-systems-labs.md` blocks implementation while it is `draft`. Creating
a lab directory, pinning dependency versions, or authoring the first lab needs
that spec moved to `accepted`. Editing and expanding the specs is open work.
