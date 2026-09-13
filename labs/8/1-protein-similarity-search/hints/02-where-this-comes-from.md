> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../../../docs/search-and-retrieval-track.md)
  — the track record where this candidate and its origination are recorded.
- [NCBI BLAST homepage](https://blast.ncbi.nlm.nih.gov/Blast.cgi) — what a
  ranked match is for: "BLAST can be used to infer functional and evolutionary
  relationships between sequences as well as help identify members of gene
  families."
- [The Statistics of Sequence Similarity Scores](https://www.ncbi.nlm.nih.gov/BLAST/tutorial/Altschul-1.html)
  — the track record's origination source and NCBI's primary description of
  the Karlin-Altschul statistics: the expected
  number of chance matches is proportional to the search space, a database
  search multiplies the pairwise expectation by `N/n` with `N` the total
  database length, and a normalized score plus the search space size is all
  significance needs. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- [NCBI BLAST FAQ](https://blast.ncbi.nlm.nih.gov/doc/blast-help/FAQ.html) —
  the vendor's own definition: the Expect value is "the number of hits one can
  'expect' to see by chance when searching a database of a particular size".
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [How BLAST E-values are calculated](https://sequenceserver.com/blog/blast-e-value-meaning/)
  — the same amount of similarity gets a weaker significance as the database
  grows, because larger databases have
  more chances of producing the alignment by chance. Solution-bearing: this
  belongs in `hints/`, never in `README.md`.
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
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [OpenSearch script score](https://docs.opensearch.org/latest/query-dsl/specialized/script-score/)
  — custom score computation applied to the documents a cheaper query already
  filtered. Solution-bearing: this belongs in `hints/`, never in
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
