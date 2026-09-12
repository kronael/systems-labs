---
status: draft
---

# Relevance evaluation service

## Brief

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

## Prepared scaffold

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

## Architecture questions

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

## Adversarial evaluation

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

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **trec_eval** is the TREC community's standard scorer: it judges a frozen run
  file against a frozen judgement file, so the engine, the corpus, and serving
  sit entirely outside its world — nothing can drift between two invocations,
  and nothing serves traffic either. See the
  [usnistgov/trec_eval](https://github.com/usnistgov/trec_eval) documentation.
- **Quepid** puts collaborative judgement collection first and evaluates its
  cases against the live engine, so judgements evolve continuously and a case's
  score follows whatever the index holds now rather than a pinned corpus
  version. See the [Quepid](https://github.com/o19s/quepid) documentation.
- **Elasticsearch's ranking evaluation endpoint** makes evaluation a stateless
  engine call: it runs the supplied rated queries against the live index and
  returns a metric, keeping no run history, so identity and comparability
  across runs are entirely the caller's problem. See the
  [ranking evaluation API reference](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/search-rank-eval).

The lab does not run them.

## Scope

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

## Code pointers

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../docs/search-and-retrieval-track.md)
  — the track record where this candidate is recorded.
- Järvelin and Kekäläinen,
  [Cumulated gain-based evaluation of IR techniques](https://dl.acm.org/doi/10.1145/582415.582418)
  (ACM TOIS 20(4), 2002;
  [PDF mirror](https://faculty.cc.gatech.edu/~zha/CS8803WST/dcg.pdf)) — defines
  the cumulated-gain family: gains from graded judgements are summed down the
  ranking, a logarithmic discount devalues documents retrieved late, and
  normalisation against the ideal ordering makes results comparable across
  queries and admits statistical testing of differences. Solution-bearing: this
  belongs in `HINTS.md`, never in `README.md`.
- [Evaluation of ranked retrieval results](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-ranked-retrieval-results-1.html)
  (Introduction to Information Retrieval, ch. 8) — the textbook treatment of
  ranked measures for graded relevance, with the logarithmic position discount
  and the normalisation factor that makes a perfect ranking score 1.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [Information retrieval system evaluation](https://nlp.stanford.edu/IR-book/html/htmledition/information-retrieval-system-evaluation-1.html)
  (Introduction to Information Retrieval, ch. 8) — states that reporting
  results obtained by tuning parameters on the same collection is wrong, that
  tuning belongs on development collections with performance reported on a
  held-out set, and that around fifty information needs is the usual minimum
  for a stable evaluation. Solution-bearing: this belongs in `HINTS.md`, never
  in `README.md`.
- Buckley, Dimmick, Soboroff, and Voorhees,
  [Bias and the limits of pooling for large collections](https://www.nist.gov/publications/bias-and-limits-pooling-large-collections)
  — judgement sets are built by judging only a pooled sample of each
  collection, unjudged documents are assumed nonrelevant, and on large
  collections the resulting judgements are systematically biased against
  systems that did not contribute to the pool. Establishes that a judgement set
  is incomplete by construction, not by accident.
- [TREC relevance judgements](https://trec.nist.gov/data/qrels_eng/index.html)
  — the qrels file is topic, iteration, document number, and relevancy, and
  documents absent from it were never judged and are assumed irrelevant in
  TREC's evaluations. Establishes what a judgement set actually is.
- [TREC overview](https://trec.nist.gov/overview.html) — NIST pools the
  participants' top-ranked results, judges the pooled documents, and publishes
  the test collections so systems can be evaluated at any later time.
- [usnistgov/trec_eval](https://github.com/usnistgov/trec_eval) — the standard
  TREC evaluation tool: a judgement file plus a run file in, more than fifty
  retrieval measures out.
- [OpenSearch ranking evaluation API](https://docs.opensearch.org/latest/api-reference/search-apis/rank-eval/)
  — the engine's `_rank_eval` endpoint runs rated queries against the live
  index, returns a metric score, and lists each query's returned-but-unrated
  documents. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- [OpenSearch keyword search](https://docs.opensearch.org/latest/search-plugins/keyword-search/)
  — scores come from BM25 over term frequency, inverse document frequency, and
  document-length normalisation, so a hit's score is a property of the corpus
  statistics as much as of the match, and moves when the corpus changes.
- Implementation pointers do not exist while the spec is `draft`.
