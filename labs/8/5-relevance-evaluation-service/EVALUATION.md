> Spoilers. Open only when stuck.

# Relevance evaluation service — how this lab is checked

Open this when you are checking your own work, or when you have finished and
want to know what a strong design holds. It states the failure schedule the
gate runs and what each scenario must leave true. Reading it before you design
hands you the boundary you were meant to find.

## The failure schedule

Faults fire at named barriers — a named query–document pair, a named run
boundary — never on a timer and never at random. Verification:

- evaluates the same candidate against the same corpus and judgement versions
  twice and requires the identical reported score;
- recomputes every reported score independently from the accepted inputs named
  in the report's lineage and requires a match;
- submits the prepared overfit candidate — built to score higher on the queries
  that shaped it and lower on the rest — and fails any harness that reports it
  as an improvement; the honest report shows the gain where the candidate was
  tuned beside its effect everywhere else;
- flips the judgement of one named query–document pair after a run has been
  reported, then requires the issued report to remain reproducible as issued,
  the flip to be visible in lineage, and any re-evaluation to name the changed
  judgement set;
- changes the corpus between two evaluation runs and fails any presentation of
  the two scores as comparable;
- issues a query for which a returned document carries no judgement at all and
  requires that document to appear in the report by identity;
- restarts the harness mid-run and requires the run ledger to contain either
  the completed run under its exact identity or no run presented as complete.

Verification does not inspect private functions and does not require a named
measure, a named division of queries, or a named statistical test. It accepts
any harness whose reports are reproducible, exact about lineage, and honest
about the overfit candidate.
