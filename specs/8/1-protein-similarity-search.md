---
status: draft
---

# Protein similarity search — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Verification queries with the planted fixtures and asserts that the true match
outranks every decoy: a design that trusts fast candidate retrieval and skips
recomputable scoring produces the inverted ranking, and verification detects it
by recomputing scores from the published rules.

Faults fire at named barriers, never on a timer and never at random:

- **Growth barrier.** The failure schedule issues a fixture query, then loads the named
  batch that carries the corpus past 20 million sequences, then issues the
  identical query again. Matches present in both answers must keep identical
  regions and scores; every confidence must be weaker in the second answer;
  any match whose confidence crosses the declared reporting threshold must be
  suppressed or flagged; and the two answers must name different corpus
  versions.
- **Restart barrier.** When the candidate phase for the named query begins,
  the controller fails one named shard of the index the query ranges over, so
  the engine answers from the shards that are still available. The service
  must return either a complete answer or an explicit error, never a quietly
  smaller answer assembled from the shards that responded.
- **Indexing barrier.** The controller makes indexing fail for one named
  accession that is planted as the top match for a fixture query. The failure
  must surface in the corpus state, and the affected answer must disclose the
  discrepancy rather than presenting a confident ranking that silently omits
  its best match.

Checks observe public boundaries only. They do not inspect private
functions, do not require a named engine feature, and compare every score
against an independent recomputation from the published scoring rules.
