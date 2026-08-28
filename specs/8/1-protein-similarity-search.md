---
status: draft
---

# Protein similarity search

## Brief

Design and build a system that serves similarity search over a growing protein
corpus: given a query sequence, it returns the matching corpus sequences,
ranked, with the region each match covers, a score, and a stated confidence,
while new sequences arrive continuously.

A match is a hypothesis about relationship: sequence similarity is how an
unknown sequence acquires a provisional function, and how members of a family
are recognized. That purpose is why a score that ignores the size of the
corpus it was drawn against is dangerous rather than merely imprecise — it
invites a claim of relationship the data does not support. No biology
background is required: a sequence is a string over a fixed twenty-letter
alphabet with a stable accession. A match is a scored region of similarity
between the query and one corpus sequence. The public contract separates two
properties that a naive design merges: the **score** of a match is a function
of the two sequences and the published scoring rules alone, so any
implementation can recompute it from the response; the **confidence** states
how often a match at least this good is expected to arise between unrelated
sequences in a corpus of this size, so it is a property of the corpus and must
move when the corpus does. Every answer names the corpus state it was computed
against.

A list of identifiers is not an answer. A reported match names the region of
both sequences it covers and carries the recomputable score; an answer whose
ranking cannot be reproduced from the published scoring rules fails the
contract even when it looks plausible.

The assignment is the whole service: ingestion path, index and store design,
query path, scoring, confidence, and end-to-end tests. How retrieval candidates
are found and turned into scored matches, how the index and the durable store
divide the work and stay honest with each other, how ingestion work is spread
across a growing corpus, and how confidence is computed are the learner's
decisions.

## Prepared scaffold

The supplied Compose stack starts OpenSearch, PostgreSQL as the durable store,
the corpus generator, the source replay for cached UniProt recordings, and the
fault controller. The generator emits seeded sequences containing planted
match families: for named query fixtures it plants one strong true match and a
decoy family that shares many short substrings with the query yet scores
poorly under the published rules.

The scaffold fixes the scoring rules as data — a substitution table over the
alphabet and gap costs — so scores are comparable across implementations and
choosing them stays outside the problem. Every sequence carries a stable
accession so verification can assert exact identities and score histories.

The learner owns the ingestion service, the index and store design, and the
query API. No cloud account is required.

## Requirements

The service exposes at most four public operations: submit a query and receive
ranked matches; fetch a stored result with its confidence and the corpus
version it reflects; append sequences to the corpus; and report corpus state —
version, sequences accepted, sequences searchable, and any discrepancy between
the two.

Ranking follows the score, with deterministic tie-breaking by accession. The
same query against the same corpus version returns the same ranked identities,
regions, and scores on every run. The confidence in an answer changes when and
only when the corpus changes.

A sequence acknowledged by ingestion either becomes searchable or has its
failure reported through the corpus state; the durable store and the search
index must not silently diverge, and an answer computed against a corpus with
a known discrepancy says so. Ingestion continues while queries run; the
service does not stop accepting sequences to make an answer correct.

The learner declares a reporting threshold on confidence — what is returned
and what is suppressed — and a latency service level for queries. Both appear
in the evidence report.

The scale target: the corpus grows from 2 million to 20 million sequences
(roughly 700 million to 7 billion residues) during the evidence run, ingestion
sustains 5,000 sequences per second, and 50 concurrent queries of 30 to 3,000
residues run throughout. These three numbers size the problem — the tenfold
growth is what moves significance while scores stay put, and a corpus that
does not grow cannot show the quirk. They are not pass thresholds; thresholds
stay relative, calibrated, structural, or learner-declared.

The evidence must contain the ingest rate sustained under query load, query
latency by query-length class against the declared service level, index and
store size against corpus size, and the trajectory of a tracked match across
corpus growth: its score and its confidence at each corpus version. How the
measurement is produced is the learner's choice; no telemetry stack is
required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what the retrieval stage actually guarantees about its candidate set, and
  what would be wrong with returning that set as the answer;
- what the score is a function of, what the confidence is a function of, and
  why the two must not be collapsed into one number;
- what an answer means while the corpus is growing: which corpus version it
  reflects, how that version is named, and what happens to a result fetched
  after further growth;
- what a reader sees between the acknowledgement of a sequence and its
  visibility in search, and how the service bounds or reports that gap;
- how the service detects an incomplete candidate set from the engine, and
  what it does instead of returning a silently smaller answer;
- which store is authoritative for corpus membership, how divergence between
  the index and the durable store is detected, and how it is repaired;
- how query cost changes as the corpus grows tenfold, and what keeps it
  bounded;
- which guarantee degrades first under sustained overload, and why that
  choice is acceptable or not.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

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
  the controller restarts one OpenSearch node. The service must return either
  a complete answer or an explicit error — never a quietly smaller answer
  assembled from the shards that survived.
- **Indexing barrier.** The controller makes indexing fail for one named
  accession that is planted as the top match for a fixture query. The failure
  must surface in the corpus state, and the affected answer must disclose the
  discrepancy rather than presenting a confident ranking that silently omits
  its best match.

Checks observe public boundaries only. They do not inspect private
functions, do not require a named engine feature, and compare every score
against an independent recomputation from the published scoring rules.

## Acceptance evidence

The same query at the same corpus version returns byte-identical ranked
identities, regions, and scores across repeated runs. The tracked match's
history shows a flat score across the full tenfold growth while its confidence
weakens version by version, including the version at which it crossed the
declared reporting threshold. Decoys never outrank the planted true match. A
node restart during a query never yields a silently reduced answer, and a
failed indexing of an acknowledged sequence is visible both in the corpus
state and in the answers it affects.

The evidence report contains the sustained ingest rate under concurrent query
load, query latency by length class against the declared service level, index
and store growth against corpus size, and the score-and-confidence trajectory
table for the tracked match. It names the point at which the design would have
to trade answer completeness for latency, and one residual limitation.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

- **NCBI BLAST+** is the reference product as a batch command: the database is
  built once and frozen, and significance is computed against that snapshot,
  so corpus growth means rebuild-and-rerun rather than a serving contract with
  named corpus versions. See the
  [BLAST Command Line Applications User Manual](https://www.ncbi.nlm.nih.gov/books/NBK279690/).
- **DIAMOND** aligns proteins at 100x to 10,000x the speed of BLAST by
  organizing work as batch analysis of big sequence data, which buys
  throughput for whole-dataset jobs and gives up the per-query serving path
  this lab is about. See the
  [DIAMOND repository](https://github.com/bbuchfink/diamond).
- **OpenSearch k-NN vector search** replaces string similarity with distance
  between embeddings; approximate search "requires a sacrifice in accuracy but
  increases search processing speeds appreciably", which moves the corpus
  dependence out of the score and into which neighbours are found at all. See
  [approximate k-NN search](https://docs.opensearch.org/latest/vector-search/vector-search-techniques/approximate-knn/).

Read their documentation on database builds, batch throughput, and recall. The
lab does not run them.

## Scope

The expected focused time is sixteen to twenty hours; this track is
deliberately denser than phases 1 to 5 because the interaction between the
search engine and the durable store is the lesson, and the redesign loop
across ingestion, index/store consistency, and significance recomputation
prices out well above a phase 1-5 lab. The learner builds the ingestion service,
the index and store design, and the query API. OpenSearch, PostgreSQL, the
generator, the scoring rules, the planted fixtures, and the fault schedules
are prepared.

CI and every required gate use the seeded generator only and run locally with
no cloud account. Real UniProt data is opt-in through `make source`: bounded
to the manually curated subset (575,503 entries at release 2026_02),
checksummed, cached under `${PREFIX:-/srv}/data/systems-labs/sources/`, and
never fetched on a request path. UniProt data is CC BY 4.0, and the copyright
statement travels with every cached copy.

Choosing scoring rules, embeddings and semantic similarity, multi-node cluster
operations, and any user interface are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/4-search-and-retrieval-track.md`](../0/4-search-and-retrieval-track.md)
  — the track record where this candidate and its origination are recorded.
- [NCBI BLAST homepage](https://blast.ncbi.nlm.nih.gov/Blast.cgi) — what a
  ranked match is for: "BLAST can be used to infer functional and evolutionary
  relationships between sequences as well as help identify members of gene
  families."
- [The Statistics of Sequence Similarity Scores](https://www.ncbi.nlm.nih.gov/BLAST/tutorial/Altschul-1.html)
  — NCBI's primary description of the Karlin-Altschul statistics: the expected
  number of chance matches is proportional to the search space, a database
  search multiplies the pairwise expectation by `N/n` with `N` the total
  database length, and a normalized score plus the search space size is all
  significance needs. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- [NCBI BLAST FAQ](https://blast.ncbi.nlm.nih.gov/doc/blast-help/FAQ.html) —
  the vendor's own definition: the Expect value is "the number of hits one can
  'expect' to see by chance when searching a database of a particular size".
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [How BLAST E-values are calculated](https://sequenceserver.com/blog/blast-e-value-meaning/)
  — the track record's origination source: the same amount of similarity gets
  a weaker significance as the database grows, because larger databases have
  more chances of producing the alignment by chance. Solution-bearing: this
  belongs in `HINTS.md`, never in `README.md`.
- [UniProt FTP license](https://ftp.uniprot.org/pub/databases/uniprot/LICENSE)
  — CC BY 4.0; the databases may be copied and redistributed freely provided
  the copyright statement is reproduced with each copy.
- [UniProt current release notes](https://ftp.uniprot.org/pub/databases/uniprot/current_release/relnotes.txt)
  — release 2026_02 holds 575,503 curated and 149,234,636 automatically
  annotated entries, which establishes the real corpus sizes and that the
  corpus grows with every release.
- [OpenSearch keyword search](https://docs.opensearch.org/latest/search-plugins/keyword-search/)
  — the engine's native relevance is BM25, a lexical score weighted by how
  common a term is across the corpus; it is not the published match score, and
  a design that returns it as one has answered the wrong question.
- [OpenSearch n-gram tokenizer](https://docs.opensearch.org/latest/analyzers/tokenizers/ngram/)
  — splits text into overlapping fixed-length substrings for partial matching,
  with `min_gram`, `max_gram`, and the `index.max_ngram_diff` bound.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [OpenSearch script score](https://docs.opensearch.org/latest/query-dsl/specialized/script-score/)
  — custom score computation applied to the documents a cheaper query already
  filtered. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- [OpenSearch refresh API](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/)
  — a document is not searchable until a refresh converts in-memory structures
  into searchable segments, which is the write-to-visibility gap the corpus
  state must account for.
- [OpenSearch search API](https://docs.opensearch.org/latest/api-reference/search-apis/search/)
  — `allow_partial_search_results` defaults to `true`, so a query that loses
  shards to an error or timeout returns a smaller answer unless the caller
  notices.
- Implementation pointers do not exist while the spec is `draft`.
