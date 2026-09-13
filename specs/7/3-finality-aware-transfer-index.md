---
status: draft
---

# Finality-aware transfer index — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Every fault fires at a named barrier through the node's own control surface,
never on a timer and never at random. The failure schedule forks the head at
a named block and mints a competing branch one block longer, so a named
transfer the index has already served returns marked `removed: true` and a
competing transfer takes its place. It repeats the fork at depth 32, past
the safe head and short of the finalized boundary. It delivers one named
transfer, waits until the fast view serves it, and withdraws it; verification
asserts that no settled answer ever contained it. The schedule kills the
indexer at a named block mid-range and restarts it, and halts finality at a named block while
the head keeps growing, then verification queries the settled view. Balance
and history queries run against both views in every window: before the fork,
inside the contested range, and after finality passes it.

Evaluation compares every answer with balances and histories computed
independently from the node's surviving chain. Counts alone prove nothing: a
settled history must contain exactly the surviving identities and no
identity from an abandoned branch. No storage layout, framework, or client
library is required.
