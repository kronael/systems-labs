---
status: draft
---

# Web crawl and index

## Brief

Design and build a system that crawls a bounded set of sites politely and
serves a search index over the result with an explicit staleness guarantee.

The product is a search API. A query returns ranked pages from the crawled
corpus, and every answer states how stale it may be: for each result, when the
system last confirmed the content it is showing. The corpus changes while the
system runs, the crawl budget cannot cover every page, and the index is
therefore stale by construction. The real product is the decision of which
pages to visit, which to revisit, how often, and what to say about the pages
the budget never reaches.

Politeness is not a courtesy. The Robots Exclusion Protocol is a published
standard, and compliance with it is a hard requirement of this lab: a
disallowed path is never fetched, ever, under any failure.

The assignment is the whole system: fetching, page identity, storage, indexing,
the revisit decision, the search API, the staleness statement, and end-to-end
tests. The prompt does not prescribe how unvisited pages are tracked, how the
next fetch is chosen, what makes two URLs one page, how repeated content is
detected, how the index is laid out, or where any piece of state lives.
Extracting text and links from fetched HTML is prepared, not learner work;
none of the difficulty is parsing.

## Prepared scaffold

The supplied Compose stack starts OpenSearch, PostgreSQL, the site harness,
and the fault controller. No cloud account is required, and no required gate
touches the live Internet.

The site harness serves twenty seeded local sites totalling 200,000 pages. It
enforces a per-host request rate and records every request it receives in a
ledger, which is the grader's ground truth. Its `robots.txt` files exercise
the standard's edges: wildcard and end-anchor rules, longest-match precedence,
groups that must be merged for the crawler's own product token, one file large
enough that a parser truncating before the standard's minimum misses a rule
that matters, and a `robots.txt` reached only through redirects. Pages carry
`ETag` and `Last-Modified` validators and answer conditional requests,
including the strictly standard-conformant case where a page changed twice
within one second keeps its `Last-Modified` and truthfully answers 304 to
`If-Modified-Since`. The harness rewrites named pages at named barriers.

A prepared extraction library converts a fetched page into indexable text and
outgoing links. The learner owns everything between the harness and the search
API: the crawl, the state, the index, and the query path.

## Requirements

The system crawls the harness sites, indexes what it fetches, and serves
full-text search over the result while the crawl continues. Queries whose
result sets run past ten thousand documents must be fully paginable, and
pagination must behave coherently while documents are being added and
replaced underneath it.

Every answer carries the staleness statement. The statement is never
optimistic: it never claims a confirmation newer than a fetch that actually
observed the content behind the result. A page that changed on the site is
eventually reflected in the index within a bound the learner declares, and the
declared bound holds in the evidence.

Politeness is binding. The crawler identifies itself with a stable product
token, honours the per-host rate the harness enforces, obeys `robots.txt` per
RFC 9309 — including complete disallow while the file is unreachable with
server errors — and does not use a cached `robots.txt` beyond the standard's
guidance. A disallowed URL never appears in the harness ledger as fetched.

The crawl survives a restart. After a kill mid-crawl, the system resumes
without refetching the corpus from the start, and every politeness obligation
holds across the boundary.

The scale target is 200,000 pages across twenty hosts, a harness-enforced
ceiling of one request per second per host, roughly six percent of the corpus
rewritten per hour at named identities, and a sustained fifty queries per
second from twenty concurrent clients while the crawl runs. These numbers size
the problem — at twenty fetches per second the budget cannot cover the corpus
inside the evidence window, so revisiting one page is always the choice not to
visit another. They are not pass thresholds; thresholds stay relative,
calibrated, structural, or learner-declared.

The evidence must show coverage over time, the distribution of page ages
behind answered queries against the declared bound, and what conditional
requests saved compared to unconditional refetching. How those measurements
are produced is the learner's choice; no telemetry stack is required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what makes two URLs the same page, at what point in the pipeline that is
  decided, and what happens to the loser;
- how the fetch budget is divided between discovering pages never seen and
  re-verifying pages already indexed, and what observation would shift that
  division;
- what a 304 response actually proves about a page, and for how long the
  design is willing to act on that proof;
- what the staleness statement means operationally — per result or per index —
  and from which recorded facts it is derived;
- what happens to a host's pages while its `robots.txt` cannot be fetched,
  and when that decision is reconsidered;
- which state survives a crash, which is reconstructed, and which fetches are
  repeated after a restart;
- how the delay between writing a document and that document becoming
  searchable enters the staleness statement;
- which pages will never be revisited under the stated budget, and what the
  product says about them.

Alternative designs must be compared. The chosen design needs stated failure
modes and one residual limitation.

## Adversarial evaluation

Faults fire at named barriers, never on a timer and never at random, so every
run produces the same history and the grader asserts exact URL identities
against the harness ledger.

- When a named page is acknowledged as indexed, the harness rewrites its
  content. The change must be reflected in search results within the declared
  bound, and no answer in between may claim a confirmation newer than the last
  fetch that saw the old content.
- At a named fetch, a host's `robots.txt` begins returning server errors.
  While it is unreachable, the ledger must show no fetch on that host beyond
  what the standard's caching guidance permits.
- A named page changes twice within one second, keeps its `Last-Modified`,
  and truthfully answers 304 to a conditional request. The changed content
  must still be reflected eventually, and the staleness statement must never
  treat that 304 as a confirmation of the bytes.
- The harness installs a redirect chain that makes two named URLs one page.
  The index must converge on one document for that page, and no query may
  return both identities.
- When a named page is acknowledged, the crawler is killed and restarted
  mid-crawl. The crawl resumes, the ledger shows no disallowed fetch and no
  wholesale refetch, and the staleness statement remains honest across the
  gap.

The grader does not inspect private functions and does not look for a named
design. It reads the harness ledger, the index history per URL, and the
answers of the public search API, and compares them against the declared
schedule of content changes.

## Acceptance evidence

A disallowed path never appears in the ledger as fetched, across the whole
run and every fault. For every named changed page, the index history shows the
old content replaced by the new within the declared bound, in fetch order.
Every staleness statement sampled during the run is no newer than the fetch
that confirmed the content behind it. The two redirected identities resolve to
one document. The restart leaves no double-indexed page and no politeness
violation.

The evidence report includes coverage of the corpus over time, the revisit
interval distribution across pages, the ratio of 304 to full responses on
revisits and the bytes that ratio saved against an unconditional baseline, the
distribution of content age behind answered queries against the declared
bound, and query behavior while paginating past ten thousand results during
active indexing. It names the pages the budget never reached and states what
the product guarantees about them, and it names one residual limitation.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **Apache Nutch** ships the entire crawl loop — pending-URL state, fetch
  scheduling, parsing, and indexing into Solr or Elasticsearch — as a
  configurable batch pipeline on Hadoop data structures, so the revisit policy
  this lab makes the learner design is a plugin selected in configuration.
- **Apache StormCrawler** treats the crawl as an unbounded low-latency stream
  on Apache Storm rather than a scheduled batch, keeping URL status in an
  external store such as OpenSearch, which moves the revisit decision into
  stream re-emission.
- **Common Crawl** removes the crawl entirely: a nonprofit's shared corpus of
  hundreds of billions of pages, grown by billions of pages a month, consumed
  as archives instead of fetched. Freshness becomes whatever the shared
  crawler's schedule delivered, and no one consumer chooses what is revisited.

Read the [Nutch](https://nutch.apache.org/),
[StormCrawler](https://stormcrawler.apache.org/), and
[Common Crawl](https://commoncrawl.org/) documentation. The lab does not run
them.

## Scope and data

The expected focused time is six to eight hours. The learner builds the crawl,
the state, the index, the search API, and the staleness statement. OpenSearch,
PostgreSQL, the site harness, the extraction library, and the fault schedules
are prepared.

Required gates use the local harness only; the live Internet is never touched.
A real crawl is opt-in through `make source` with an explicit TOML config
naming the permitted sites: it identifies itself with a product token and
contact URL, obeys `robots.txt` and rate limits, is bounded by page and byte
count, and writes a checksummed recording under
`${PREFIX:-/srv}/data/systems-labs/sources/` that replays offline. No live
fetch ever sits on a request path, and no required gate needs a cloud
account.

JavaScript rendering, ranking quality, HTML parsing, distributed crawling,
and full-web scale are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../0/4-search-and-retrieval-track.md`](../0/4-search-and-retrieval-track.md)
  — the track record where this candidate and its origination are recorded.
- [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309.html) — the Robots
  Exclusion Protocol: a cached `robots.txt` should not be used beyond 24 hours
  (§2.4), parsers must handle at least 500 KiB (§2.5), server errors mean
  complete disallow while unavailable-status codes mean allow (§2.3.1), and
  matching is longest-match with allow as the default (§2.2.2).
- [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) — HTTP conditional
  requests: `If-None-Match` uses weak comparison and yields 304 for GET
  (§13.1.2), it takes precedence over `If-Modified-Since` (§13.2.2), and
  `Last-Modified` is implicitly weak because one-second resolution cannot
  distinguish two changes within the same second (§8.8.2.2) — which is why a
  truthful 304 is not proof the bytes are unchanged.
- [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html) — HTTP caching:
  freshness lifetime and the heuristic of a fraction of the age since
  `Last-Modified` (§4.2.2), and updating a stored response on 304 (§4.3.4).
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [OpenSearch refresh](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/)
  — an indexed document becomes searchable only after a refresh, which runs
  every second by default, so the index lags its own writes and that lag is
  part of any honest staleness statement.
- [OpenSearch pagination](https://docs.opensearch.org/latest/search-plugins/searching-data/paginate/)
  — result windows cap at ten thousand documents by default, and paginating
  deeper or consistently while the index changes requires the mechanisms this
  page names. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
