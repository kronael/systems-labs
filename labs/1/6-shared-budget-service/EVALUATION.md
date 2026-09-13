> Spoilers. Open only when stuck.

# Shared budget service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

The failure schedule drives many clients at one contended budget set at named
claim identities. It accepts a limit change at the exact moment a named
claim's decision is in flight; it kills the application process after a named
claim is decided and before its answer leaves; it restarts PostgreSQL while
decisions are in flight; it repeats submissions of already-decided claim
identities; it introduces a set of claims constructed so that their decisions
conflict pairwise and cannot all proceed; it holds a statement read open
across a burst of claims and limit changes; and it raises the overlap share
until refused work dominates and holds it there.

At the end every budget and every retained claim is read. No budget is
overspent at any point in the recorded history, every decision returned to a
client is present and unchanged, every accepted claim identity carries exactly
one decision, and every settled claim's committed amount matches its
settlement.

Checks observe only HTTP, SQL-visible state, exact request and decision
histories, process lifecycle, query plans, metrics, and the submitted
evidence. They do not require a particular schema, transaction structure, or
coordination primitive.

## What a weak design gets wrong

A simple design fails this lab at its scale target: one process
deciding claims one at a time holds the invariant but cannot hold 2,000
submissions per second from 500 clients, while a design that decides claims
concurrently on what it has read has its work handed back at a rate that
climbs with concurrency, so its delivered rate stops following its offered
rate well below the target.
