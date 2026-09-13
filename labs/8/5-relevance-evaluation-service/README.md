# Relevance evaluation service

Design and build a system that serves ranked search over a document corpus and
decides whether a proposed change to the ranking is an improvement. The system
answers search queries with ranked results, accepts graded judgements of
query–document pairs from assessors, accepts candidate ranking configurations,
and issues evaluation reports that state whether a candidate improves on the
incumbent ranking and on what evidence.

The belief this lab falsifies: a ranking change that looks better is better.
The engine attaches a score to every hit, and the score moves when the
configuration changes; none of that is evidence of improvement. A measurement
made against the same queries that shaped a change reports progress that does
not survive contact with new queries. The product here is the harness that can
tell the difference — the ranking it serves is the subject under measurement,
not the achievement.

The assignment is the whole service: serving path, judgement intake, evaluation
runs, run lineage, verdicts, reports, and end-to-end tests. The quality
measure, how judged queries are divided between shaping a change and judging
it, how a real difference between two rankers is told from noise, and what a
report must contain to be trusted are the lab's actual questions, and
`ARCHITECTURE.md` must defend them.

## What you are given

The supplied Compose stack starts OpenSearch, PostgreSQL, the corpus and
judgement generator, and the fault controller. The generator produces a seeded
document corpus, a query stream, a graded judgement set, and a set of candidate
ranking configurations whose true effects are known to verification and hidden
from the harness: at least one genuine improvement, at least one change that
reorders results without helping anyone, and at least one built to help exactly
the queries it was tuned against and nothing else. Candidates exercise the
engine's ranking surface — text analysis, per-field weighting, and the scoring
function — so a candidate is a real change to how the engine ranks, not a label.

Every document, query, judgement, and candidate carries a stable identity, so
verification can assert exact histories rather than counts. The learner owns the
service, its state design in the durable store, and the report format. No cloud
account is required.

## Requirements

The service answers ranked queries over the corpus, and keeps answering them
while evaluation runs are in progress.

Judgements arrive continuously as graded verdicts on identified query–document
pairs. A judgement recorded after a report was issued must not silently change
that report.

Given a candidate ranking configuration, the service produces a verdict:
improvement, no improvement, or cannot say — with the evidence the verdict
rests on. A verdict of improvement must be supported by queries that did not
influence the candidate, and the report must state which queries influenced it.

Reports are reproducible: the same candidate evaluated against the same corpus
and the same judgement set yields the same score, every time. Every reported
number names the exact corpus, ranker configuration, and judgement set behind
it, and two scores are presented as comparable only when that lineage says they
are.

Reports are honest about coverage: a scored result that has no judgement at all
must appear in the report by identity, and the verdict must state how much of
what it scored was actually judged. The engine's per-hit score is computed by
its ranking function from term and corpus statistics; it is not a judgement,
and no verdict may rest on it.

The scale target is 500,000 generated documents, 2,000 judged queries carrying
roughly 100,000 graded judgement pairs, sustained serving of 100 queries per
second while a full evaluation run replays the entire judged query set, ten
candidates evaluated back to back, and a ledger of at least 200 evaluation runs
that remains queryable with full lineage. These numbers size the problem: the
query set is large enough that a real difference between two rankers can be
distinguished from per-query noise even after part of the set has been spent
shaping a change, and the corpus is large enough that its statistics move when
it changes. They are not pass thresholds; thresholds stay relative, calibrated,
structural, or learner-declared.

The evidence must contain the verdict on each prepared candidate, a
demonstration of score reproducibility, the coverage statement of each report,
and serving latency against the learner's declared service level while an
evaluation runs. How those numbers are produced is the learner's choice; no
telemetry stack is required.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- which property of a ranking the chosen quality measure rewards, why that is
  the property this product needs, and which regressions the measure is blind
  to;
- how graded judgements enter the measure, and what the measure reports when a
  scored result has no judgement;
- how the judged queries are divided between shaping a change and judging it,
  what that division costs in sensitivity, and when the judging portion is
  spent — that is, at what point queries that have judged one candidate stop
  being able to judge the next;
- what makes two reported scores comparable, and which change — corpus,
  configuration, or judgement set — breaks comparability;
- how the harness distinguishes a real difference between two rankers from
  noise across queries, and what "cannot say" means operationally;
- where the engine's per-hit score ends and the harness's measure begins, given
  that the engine's score moves with corpus statistics under an unchanged
  configuration;
- what an issued report promises after the judgement set changes, and what
  re-evaluating an old candidate against new judgements means for the old
  verdict;
- what exists after a harness restart mid-run, and what must never exist.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Acceptance evidence

The prepared candidates receive correct verdicts: the genuine improvement is
accepted with its evidence, the no-op is not claimed as progress, and the
overfit candidate is rejected with both of its numbers shown — the gain on the
queries that shaped it and the effect beyond them.

A rerun of any reported evaluation reproduces its score exactly. Every entry in
the 200-run ledger resolves to the exact corpus, configuration, and judgement
versions behind it, including the runs on either side of the judgement flip and
the corpus change. The unjudged document appears by identity in its report. The
mid-run restart leaves no partial run presented as a result.

The evidence report includes serving latency by class against the declared
service level while a full evaluation run is in progress, the cost of an
evaluation run relative to the learner's recorded baseline on the same host,
and the coverage statement of every issued verdict. It closes by naming the
improvement claim this harness still cannot check.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`hints/`, because saying it here would point straight at the answer.

- **trec_eval** — [documentation](https://github.com/usnistgov/trec_eval)
- **Quepid** — [documentation](https://github.com/o19s/quepid)
- **Elasticsearch's ranking evaluation endpoint** — [documentation](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/search-rank-eval)

## What is outside the problem

The expected focused time is twenty to twenty-four hours, denser than a phase
1-5 lab by design: the interaction between the engine and the durable store is
the lesson, and the statistical rigor a defensible verdict requires — real
difference from noise, reproducible lineage across a 200-run ledger, an
overfit candidate that must be caught rather than rewarded — prices out well
above the gates alone. The learner builds the service; OpenSearch, PostgreSQL, the generator,
and the fault schedules are prepared, and every required gate runs locally with
no cloud account.

CI uses the generated seeded corpus, queries, judgements, and candidates only.
Real public judgement sets — NIST publishes TREC judgement files — and their
document collections are opt-in, bounded, checksummed, cached under
`${PREFIX:-/srv}/data/systems-labs/sources/`, never fetched on a request path,
and never redistributed by this repository. Several TREC document collections
carry their own licences and access terms even where the judgement files are
public, so any adopted collection's terms are recorded in the research ledger
before use.

Judgement-collection interfaces, online experiments on live traffic,
learning-to-rank model training, and multi-node cluster operation are outside
the problem.

Stuck? See `hints/`.
