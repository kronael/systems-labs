> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../../../docs/search-and-retrieval-track.md)
  — the track record where this candidate is recorded.
- Järvelin and Kekäläinen,
  [Cumulated gain-based evaluation of IR techniques](https://dl.acm.org/doi/10.1145/582415.582418)
  (ACM TOIS 20(4), 2002;
  [PDF mirror](https://faculty.cc.gatech.edu/~zha/CS8803WST/dcg.pdf)) — defines
  the cumulated-gain family: gains from graded judgements are summed down the
  ranking, a logarithmic discount devalues documents retrieved late, and
  normalisation against the ideal ordering makes results comparable across
  queries and admits statistical testing of differences. Solution-bearing: this
  belongs in `hints/`, never in `README.md`.
- [Evaluation of ranked retrieval results](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-ranked-retrieval-results-1.html)
  (Introduction to Information Retrieval, ch. 8) — the textbook treatment of
  ranked measures for graded relevance, with the logarithmic position discount
  and the normalisation factor that makes a perfect ranking score 1.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [Information retrieval system evaluation](https://nlp.stanford.edu/IR-book/html/htmledition/information-retrieval-system-evaluation-1.html)
  (Introduction to Information Retrieval, ch. 8) — states that reporting
  results obtained by tuning parameters on the same collection is wrong, that
  tuning belongs on development collections with performance reported on a
  held-out set, and that around fifty information needs is the usual minimum
  for a stable evaluation. Solution-bearing: this belongs in `hints/`, never
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
  documents. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- [OpenSearch keyword search](https://docs.opensearch.org/latest/search-plugins/keyword-search/)
  — scores come from BM25 over term frequency, inverse document frequency, and
  document-length normalisation, so a hit's score is a property of the corpus
  statistics as much as of the match, and moves when the corpus changes.
- Implementation pointers do not exist while the spec is `draft`.
