# Protein similarity search

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

## What you are given

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

## What your ARCHITECTURE.md must explain

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

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does
not run them. What each does differently at this lab's boundary is in
`HINTS.md`, because saying it here would point straight at the answer.

- **NCBI BLAST+** — [documentation](https://www.ncbi.nlm.nih.gov/books/NBK279690/)
- **DIAMOND** — [documentation](https://github.com/bbuchfink/diamond)
- **OpenSearch k-NN vector search** — [documentation](https://docs.opensearch.org/latest/vector-search/vector-search-techniques/approximate-knn/)

Stuck? See `HINTS.md`.
