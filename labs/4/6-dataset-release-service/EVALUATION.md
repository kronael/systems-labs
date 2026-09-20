> Spoilers. Open only when stuck.

# Dataset release service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Every fault fires at a named barrier, never on a timer and never at random.
Each barrier is the store's acknowledgement of a write the service made, held
at the transport layer in front of the store, so every design reaches it
whatever its layout.

- **Interrupted release.** When the acknowledgement arrives for the write that
  carries release R2 past 9 GiB accepted, the controller holds that
  acknowledgement, kills the publisher process, restarts it, and then releases
  the acknowledgement.
- **Stale publisher.** When publisher A's acknowledgement arrives for the
  write that first makes release R3 readable at its address, the controller
  holds it, starts publisher B, lets B publish R3 and the increment that
  follows it, and only then releases A's acknowledgement.
- **Reader between keys.** When the acknowledgement arrives for the first
  store write of increment N, the controller holds it while the eight
  consumers keep reading current and fetching what it names. A variant kills
  the service at the same barrier and restarts it.

## What must remain true

After the interrupted release, R2 is current and byte-exact, and the bytes
admitted for R2 after the restart do not exceed the bytes that were still
unacknowledged at the kill plus one unit of the learner's declared size. A
design that starts again from the beginning sends 12 GiB against a bound near
3 GiB, and fails on that number rather than on taste.

After the stale publisher, every read of current taken by every consumer is
non-decreasing, and the final current names R3 with the increment that
followed it. A release A believed it was publishing may not become current
after B has moved past it.

Through the reader scenario, no consumer that read current and fetched what it
named received an absent or partial answer, and no consumer observed an
increment number go backwards. After the restart variant, current names
increment N only once N is fetchable and byte-exact.

At the end of every run the store holds no bytes for work the service will
never publish, or the report names those bytes and says why they remain. A run
in which no release exceeds the single-request limit fails, because the regime
the lab is about was never entered.

## What this gate does not prove

The local store is not the hosted service, and this lab has no `make smoke` to
compare them, so these claims rest on the emulator rather than on the service
they model, and a design must not lean on any of them:

- which of two concurrent completions of one key wins, which is why the
  schedule holds acknowledgements so that two store writes never overlap;
- the atomicity of a conditional write against a concurrent one;
- that unfinished work never expires, which holds locally only because the
  lab disables the store's cleanup and starts no lifecycle worker;
- the two 5 GB limits, which the controller supplies rather than the store;
- what listing the parts of a finished publication returns.

## What a weak design gets wrong

A weak design keeps the knowledge needed to resume a publication inside the
process that is publishing. The kill takes it, the restart has nothing to
resume from, and the only way forward is to send the release again — which the
admitted budget turns into a number on the report.

A weak design reads the store's own listing to decide what it already sent.
The store's instruction is the opposite: keep your own record of what you sent
and what each piece was, and use the listing only for verification.

A weak design lets a publisher that was interrupted finish later and make its
release current after another publisher has moved past it, which regresses the
public read.

A weak design counts a release as published because the bytes were accepted. A
strong design distinguishes bytes the store holds from a release a reader can
fetch, accounts for the difference, and can say at any moment which of the two
every byte in the store belongs to.
