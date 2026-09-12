---
status: reference
---

# Search, retrieval, and spatial track

## Decision

Phase 8 is a fourth catalog built on retrieval. Its labs combine a search
engine with a domain store and real public corpora, and each one is deliberately
denser than a phase 1 to 4 lab: the interaction between the technologies is the
lesson, not any one of them alone.

This is the track that admits **OpenSearch**, which the core catalog excludes.
It earns entry here because retrieval is the subject rather than a second
representative of an existing category.

The domains — proteins, news, the web, maps — are chosen for the technologies
they force, not for the tasks themselves. A learner does not need biology to
discover that a similarity score is a property of the corpus rather than of the
match.

All five candidates now have full specs. They are unscored, and they enter the
catalog only after the core catalog is `accepted`.

## Origination

Every source is *cited*: no prose, code, fixture, or test is copied.

| # | Candidate | Quirk it falsifies | Origination |
|---|-----------|--------------------|-------------|
| 8/1 | Protein similarity search | A match score is a property of the match | NCBI's own account of [the statistics of sequence similarity scores](https://www.ncbi.nlm.nih.gov/BLAST/tutorial/Altschul-1.html) gives E = Kmn·e^(−λS), so a database E-value multiplies by N/n while the bit score of the identical alignment does not move; the [BLAST FAQ](https://blast.ncbi.nlm.nih.gov/doc/blast-help/FAQ.html) defines the Expect value as the number of hits expected by chance in a database of a particular size. Corpus: [UniProt](https://ftp.uniprot.org/pub/databases/uniprot/LICENSE), CC BY 4.0 |
| 8/2 | News aggregation and curation | Two stories are either duplicates or they are not | Broder defines resemblance as a value in [0,1] over shingle sets, so any duplicate boundary is a chosen point on a continuum ([Identifying and Filtering Near-Duplicate Documents](https://cs.brown.edu/courses/cs253/papers/nearduplicate.pdf)); Manku, Jain, and Das Sarma show the distance parameter trades false positives against false negatives with no natural sharp value ([Detecting Near-Duplicates for Web Crawling](https://research.google.com/pubs/archive/33026.pdf)). [RSS 2.0](https://www.rssboard.org/rss-specification) makes item `pubDate` and `guid` optional with uniqueness left to the feed source; [RFC 4287](https://www.rfc-editor.org/rfc/rfc4287) fixes `atom:id` as immutable across republication while `atom:updated` moves only for changes the publisher considers significant, so content changes under a stable identifier |
| 8/3 | Web crawl and index | A crawl produces an index of the web as it is now | [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.pdf) standardizes robots.txt: crawlers should not use a cached copy beyond 24 hours and must parse at least 500 KiB. Conditional requests and recrawl scheduling decide freshness, and the index is stale by construction |
| 8/4 | Spatial query service | A spatial index answers the spatial question | The `&&` operator [tests only 2D bounding boxes](https://postgis.net/docs/geometry_overlaps.html), and [spatial indexes index bounding boxes, not features](https://postgis.net/workshops/postgis-intro/indexing.html) — an index-only count returns 49,821 where the exact answer is 26,718. [`ST_Distance`](https://postgis.net/docs/ST_Distance.html) on SRID 4326 geometry returns degrees, [a nonsense number](https://postgis.net/workshops/postgis-intro/geography.html); [EPSG:3857](https://epsg.io/3857) is "not a recognised geodetic system", made for web mapping rather than measurement; per [RFC 7946 §5.2](https://datatracker.ietf.org/doc/html/rfc7946#section-5.2) an antimeridian-spanning bounding box has a west edge greater than its east edge. Corpus: [OpenStreetMap](https://www.openstreetmap.org/copyright), ODbL |
| 8/5 | Relevance evaluation service | A ranking change that looks better is better | Tuning on the judged set overstates performance: reporting results tuned on the same collection is wrong, and tuning belongs on development collections with a held-out test set ([IIR ch. 8](https://nlp.stanford.edu/IR-book/html/htmledition/information-retrieval-system-evaluation-1.html)). Judgement sets are incomplete by construction: pooling judges a sample, assumes the rest nonrelevant, and biases against systems that did not contribute ([Buckley et al., NIST](https://www.nist.gov/publications/bias-and-limits-pooling-large-collections); [TREC qrels](https://trec.nist.gov/data/qrels_eng/index.html)). Metric: [Järvelin and Kekäläinen, ACM TOIS 2002](https://dl.acm.org/doi/10.1145/582415.582418). Engine surface: [OpenSearch `_rank_eval`](https://docs.opensearch.org/latest/api-reference/search-apis/rank-eval/) |

## Expanded candidates

### 8/1 Protein similarity search — specced

Serve similarity search over a protein corpus: given a query sequence, return
ranked matches with scores and a stated confidence.

The tempting design indexes k-mers in a search engine and calls the retrieval
result the answer. It is not. An inverted index gives fast candidate retrieval
and no alignment, so the ranking is wrong in a way that looks plausible. The
learner must add scoring, and then meets the real surprise: the significance of
a hit depends on the size of the corpus it was found in, so growing the
database makes yesterday's confident match insignificant while its bit score
stays exactly the same.

Technologies: OpenSearch, an alignment stage, a durable store, UniProt data.

Spec: [`../8/1-protein-similarity-search.md`](../specs/8/1-protein-similarity-search.md).

### 8/2 News aggregation and curation — specced

Ingest many feeds, group items that report the same event, and serve a curated
ranked view with a stated freshness.

There is no threshold at which two stories become duplicates. Wire copy,
syndicated reprints, and follow-ups sit on a continuum, so the clustering
decision is a product decision with a measurable cost in both directions.
Publication timestamps from feeds are unreliable and sometimes move backwards,
and an article edited in place changes content under a stable identifier — so
the system's own history is not the publisher's history.

Technologies: OpenSearch, near-duplicate detection, a durable store, feed
ingestion.

Spec: [`../8/2-news-aggregation-service.md`](../specs/8/2-news-aggregation-service.md).

### 8/3 Web crawl and index — specced

Crawl a bounded set of sites politely and serve a search index over the result
with an explicit staleness guarantee.

The index is stale by construction, so the product is really the recrawl
schedule: which pages to revisit, how often, and what to do with a crawl budget
that cannot cover everything. Politeness is not optional and is now a standard.
Canonicalization decides whether two URLs are one page, and conditional
requests decide whether a revisit costs anything at all.

Technologies: OpenSearch, a crawl scheduler, a durable store, HTTP caching.

Spec: [`../8/3-web-crawl-and-index.md`](../specs/8/3-web-crawl-and-index.md).

### 8/4 Spatial query service — specced

Serve spatial queries over a real map extract: what is inside this area, what
is near this point, and what changed since a given time.

A spatial index is a filter, not an answer: the bounding-box operator returns
candidates and the exact predicate must still run, so a design that trusts the
index returns confidently wrong results. Coordinate systems finish the lesson —
degrees are not metres, web mercator distorts away from the equator, and a
bounding box that crosses the antimeridian inverts.

Technologies: PostGIS, OpenSearch geo queries, vector tiles, an OpenStreetMap
extract.

Licensing note: OpenStreetMap data is ODbL, which carries share-alike
obligations for derived databases. Extracts are opt-in, bounded, cached, and
never redistributed by this repository, exactly as the RIPE and Kraken
recordings are handled.

Spec: [`../8/4-spatial-query-service.md`](../specs/8/4-spatial-query-service.md).

### 8/5 Relevance evaluation service — specced

Serve a ranking, and prove that a change to it is an improvement.

The product is the evaluation harness: a held-out judgement set, a proper
metric, and a lineage between runs. The failure is subtle and common — a metric
tuned against the same set it is judged on reports progress that does not
survive contact with new queries.

Technologies: OpenSearch, an evaluation harness, a durable store.

Spec: [`../8/5-relevance-evaluation-service.md`](../specs/8/5-relevance-evaluation-service.md).

## Scale contract

Each lab carries a speed, a load, and an amount, per the
[lab brief contract](../specs/01-systems-labs.md#lab-brief-contract). The amount is
what forces the quirk here: 8/1 needs a corpus large enough that significance
shifts, 8/3 needs more pages than the crawl budget covers, and 8/4 needs an
extent large enough that the index cannot be scanned.

## Governing references

- [`specs/01-systems-labs.md`](../specs/01-systems-labs.md) — course-wide learning,
  evidence, source, cost, and repository contracts.
- [`lab-selection.md`](lab-selection.md) — the scored selection that
  produced the original core ten.
- [`low-level-track.md`](low-level-track.md) — the Rust and C track.
- [`blockchain-track.md`](blockchain-track.md) — the Solana and Ethereum
  track.
- [`specs/index.md`](../specs/index.md) — authoritative list and lifecycle status.
