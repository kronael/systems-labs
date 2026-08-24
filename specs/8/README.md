---
status: reference
---

# Phase 8 — Search, retrieval, and spatial

## What this phase is about

Phase 8 is a separate catalog built on retrieval: every lab pairs a search
engine with a domain store over real public corpora — proteins, news, the
web, maps — and each is deliberately denser than a phase 1–5 lab, because
the interaction between the search engine and the domain store is the
lesson, not either system alone. This is the track that admits OpenSearch,
which the core catalog excludes; it earns entry here because retrieval is
the subject rather than a second representative of an existing category.
Each candidate is grounded in a documented public source recorded in the
[track record](../0/4-search-and-retrieval-track.md).

## The labs

- [Protein similarity search](1-protein-similarity-search.md) — a service
  that answers similarity queries over a growing protein corpus with ranked
  matches, a score, and a stated confidence.
- [News aggregation service](2-news-aggregation-service.md) — a system that
  ingests many news feeds, groups items that report the same event, and
  serves a curated ranked view with a stated freshness.
- [Web crawl and index](3-web-crawl-and-index.md) — a system that crawls a
  bounded set of sites politely and serves a search index over the result
  with an explicit staleness guarantee.
- [Spatial query service](4-spatial-query-service.md) — a service that
  answers spatial questions over a prepared map extract: what is inside an
  area, what is near a point, and what changed since a given time.
- [Relevance evaluation service](5-relevance-evaluation-service.md) — a
  system that serves ranked search and decides whether a proposed change to
  the ranking is an improvement, on stated evidence.

## The technologies

**OpenSearch** is the search engine of all five labs and the reason the
phase exists. The learner meets its advanced surface — text analysis and
custom scoring, geo queries, deep pagination, and the refresh boundary
between a write and its visibility — because the requirements are set so
that surface cannot be avoided. The phase chooses it for the gap it makes
teachable: a retrieval engine answers fast with candidates, and the
distance between that answer and a correct one is what these labs measure.
Documentation: [docs.opensearch.org](https://docs.opensearch.org/latest/).

**PostgreSQL** is the durable store beside the engine in four of the five
labs. The phase chooses the pairing because the two stores make different
promises about what an accepted write means and when it can be seen, and
keeping them honestly consistent — and saying so when they diverge — is
the recurring pressure of the whole catalog. Documentation:
[postgresql.org/docs](https://www.postgresql.org/docs/).

**PostGIS** is the spatial extension to PostgreSQL and the domain store of
the spatial lab, where the map extract arrives already loaded. The phase
chooses it as the reference implementation of spatial SQL: geometry as a
first-class column type, spatial indexing, and measurement over an
explicit earth model. Documentation:
[postgis.net/documentation](https://postgis.net/documentation/).

**Docker Compose** carries the scaffold, as in every phase: the engine and
the stores, the corpus and feed harnesses, the generators, and the fault
controller. Documentation:
[docs.docker.com/compose](https://docs.docker.com/compose/).

**The corpora** are real and licensed: UniProt protein sequences
([CC BY 4.0](https://ftp.uniprot.org/pub/databases/uniprot/LICENSE)),
OpenStreetMap extracts
([ODbL](https://www.openstreetmap.org/copyright), share-alike and never
redistributed by this repository), real news feeds, a bounded real crawl,
and NIST's public
[TREC relevance judgements](https://trec.nist.gov/data/qrels_eng/index.html).
CI never touches any of them: generated seeded input is the only CI input,
and every real corpus is opt-in through `make source` — bounded,
checksummed, cached under the shared data prefix, and never fetched on a
request path.

## What this phase does not use, and why that is interesting

Each lab's `Neighbouring systems` section names what a practitioner would
have reached for instead — NCBI BLAST+, DIAMOND, k-NN vector search,
PostgreSQL full-text search, Apache Solr, pgvector, Apache Nutch,
StormCrawler, Common Crawl, MongoDB, H3, Tile38, trec_eval, Quepid,
Elasticsearch's ranking evaluation endpoint — and the one thing each does
differently at that lab's boundary. They appear as reading rather than as
dependencies: the labs never run them, and knowing what each trades away is
part of defending a design.

## Cloud

This phase needs no cloud account and touches no cloud service: every
required gate runs entirely on local OpenSearch and its domain stores in
Compose. The real corpora are downloaded once, opt-in, bounded,
checksummed, and cached — never fetched on a request path — and no
optional step uses a hosted service. See
[cloud access](../../docs/cloud-access.md).
