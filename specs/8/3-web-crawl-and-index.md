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
tests. The state that tracks crawl progress, the scheduling of the next fetch
against the declared budget, what makes two URLs one page, and the layout of
the index and the durable store are the learner's decisions. Extracting text
and links from fetched HTML is prepared, not learner work; none of the
difficulty is parsing.

## Prepared scaffold

The supplied Compose stack starts OpenSearch, PostgreSQL, the site harness,
and the fault controller. No cloud account is required, and no required gate
touches the live Internet.

The site harness serves twenty seeded local sites totalling 200,000 pages and
enforces a per-host request rate. The rate is a grant, not a constant: a
request arriving over it is refused with 429 and a `Retry-After` naming the
wait, an allowance can fall below the ceiling and climb back, and a host can
withdraw from the crawler's product token for a time, refusing every request
under that identity the same way — and a grant can change for reasons that
are not the crawler's conduct. The harness's `robots.txt` files exercise the
standard's edges: wildcard and end-anchor rules, longest-match precedence,
groups that must be merged for the crawler's own product token, one file
large enough to exercise the standard's minimum parse size, and a
`robots.txt` reached only through redirects. Pages carry `ETag` and
`Last-Modified` validators and answer conditional requests. The harness
rewrites named pages at named barriers, and the prepared schedules fix the
evidence window and the overlap window whose demands — updates to
already-served pages, `robots.txt` requests, post-error retries — a declared
freshness bound must cover.

Every request the harness receives lands in a ledger, the ground truth
verification checks against. A ledger record carries the timestamp, the URL,
the product token presented, the conditional headers presented, and the
response served — status and headers, any `Retry-After` included — and the
harness records beside it every change to a host's granted allowance.

A prepared extraction library converts a fetched page into indexable text and
outgoing links. The learner owns everything between the harness and the search
API: the crawl, the state, the index, and the query path.

## Requirements

The system crawls the harness sites, indexes what it fetches, and serves
full-text search over the result while the crawl continues. Queries whose
result sets run past ten thousand documents must be fully paginable while
documents are being added and replaced underneath them, and what a client
walking the full result set is promised — what it can never see twice, what
it can never miss — is a contract the learner declares and the evidence shows
holding.

Every answer carries the staleness statement. The statement is never
optimistic: it never claims a confirmation newer than a fetch that actually
observed the content behind the result. The declared staleness bound binds
every page the index serves: a served page that changed on the site is
reflected again within the bound, and the bound holds in the evidence. The
bound is not free: `ARCHITECTURE.md` derives it from the numbers the harness
publishes — the ceiling, the corpus, the rewrite rate — and it fits inside
the evidence window, because a bound the window cannot witness cannot hold in
the evidence. Pages the budget has never reached sit outside the bound, and
what the product promises about them is part of the product.

The declaration is constrained, not merely derived. Let F be the declared
bound; let K, R, and E count the updates to already-served pages, the
`robots.txt` requests, and the post-error retries that a prepared overlap
window requires before F expires, each of them positive; let U count the
eligible unfetched pages competing for the same allowance, and A(F) the host's
available request slots under the declaration. The declaration is admissible
only when K + R + E <= A(F) < K + R + E + U, so the work the bound promises
fits and the corpus does not. At a constant grant g that reads
(K + R + E) / g <= F < (K + R + E + U) / g; under the fault windows the
recorded allowance supplies A(F).

Politeness is binding. The crawler identifies itself with a stable product
token, honours the per-host rate the harness enforces, obeys `robots.txt` per
RFC 9309 — including complete disallow while the file is unreachable with
server errors — and does not use a cached `robots.txt` beyond the standard's
guidance. A disallowed URL never appears in the harness ledger as fetched.

Breaching the ceiling is never free, and the grant is not the crawler's to
assume: a host may cut its allowance or withdraw entirely, provoked or not,
and the obligations do not change with the reason. A named wait is binding, a
reduced allowance is the allowance, and a withdrawn host grants nothing at
all. Every step recovers — the allowance climbs back to the ceiling, the
withdrawn host returns — and the system must come back with it: after each
recovery the crawl spends the host's grant as it did before, and the run ends
with every host granting the full rate.

The allowance is one budget, not three. Content fetches, `robots.txt`
refetches under the standard's caching guidance, and retries after error
responses all draw on the same per-host rate; the ledger counts them alike,
and nothing is exempt. When the allowance cannot cover all of them, the
system decides what it stops spending on first — and while a host's
`robots.txt` is unreachable and no cached copy the standard permits answers
for it, a content fetch on that host breaks the standard, not merely the
budget.

The crawl survives a restart. After a kill mid-crawl, the system resumes
without refetching the corpus from the start, and every politeness obligation
holds across the boundary.

The scale target is 200,000 pages across twenty hosts, a harness-enforced
ceiling of one request per second per host, roughly six percent of the corpus
rewritten per hour at named identities, and a sustained fifty queries per
second from twenty concurrent clients while the crawl runs. These numbers size
the problem — one uninterrupted pass over the corpus at the ceiling costs
200,000 / 20 = 10,000 seconds, and the evidence window is shorter than that,
so the budget cannot cover the corpus inside it, revisiting one page is always
the choice not to visit another, and an allowance lost to a breach is pages
the crawl never gets back. They are not pass thresholds; thresholds stay
relative, calibrated, structural, or learner-declared.

The evidence must show coverage over time, the distribution of page ages
behind answered queries against the declared bound, what re-verifying content
that had not changed cost, and how each host's allowance was spent across
content fetches, `robots.txt` fetches, and retries. How those measurements
are produced is the learner's choice; no telemetry stack is required.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- what makes two URLs the same page, at what point in the pipeline that is
  decided, and what happens to the loser;
- how the fetch budget is divided between discovering pages never seen and
  re-verifying pages already indexed, and what observation would shift that
  division;
- which kind of work is shed first when one host's allowance cannot cover
  content fetches, `robots.txt` refetches, and retries at once, and what
  observation would change that order;
- what a 304 response actually proves about a page, and for how long the
  design is willing to act on that proof;
- from which recorded facts the staleness statement is derived;
- what happens to a host's pages while its `robots.txt` cannot be fetched,
  and when that decision is reconsidered;
- what the system does between a host's first refusal and its recovery — what
  it stops sending, and how it learns the host is back;
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
run produces the same history and verification asserts exact URL identities
against the harness ledger.

- When a named page is acknowledged as indexed, the harness rewrites its
  content. The change must be reflected in search results within the declared
  bound, and no answer in between may claim a confirmation newer than the last
  fetch that saw the old content.
- At a named fetch, a host's `robots.txt` begins returning server errors; at
  a later named fetch it recovers. While it is unreachable, the ledger must
  show no fetch on that host beyond what the standard's caching guidance
  permits, and the refetches those errors invite must stay inside the host's
  one allowance: the window must end with the full ceiling intact.
- When a named fetch on a named host is acknowledged, the harness cuts that
  host's allowance to half the rate the crawler itself held over a window the
  schedule fixes, and answers anything faster with 429 and a `Retry-After`
  naming the wait — the cut chases the crawler's own cadence, so every design
  that spends its budget meets it. The ledger must show at least one 429 on
  the host and, after each, no request before its named wait has passed; once
  a named count of requests has arrived on time the full ceiling is restored,
  the host's named rewritten pages must still be reflected within the
  declared bound, and the crawler's cadence on the host must return to what
  it held before the cut, within a precision the learner declares.
- While that allowance is cut, at a named fetch the same host's `robots.txt`
  begins returning server errors; at a later named fetch it recovers. The
  content fetches a cached copy still permits, the refetches the errors
  invite, and the retries the named waits schedule now contend inside half an
  allowance. The ledger must show the host's whole traffic inside it, no
  request before its named wait, and the host's named rewritten pages
  reflected within the declared bound after both recoveries.
- When a named page is acknowledged as indexed, the harness withdraws its host
  from the crawler's product token — every request under that identity, on
  any connection, is refused with 429 and a `Retry-After` naming the wait —
  and restores it once a named count of requests has landed on the other
  hosts; a refused request at the withdrawn host does not postpone the
  restoration, but its named wait binds like any other. While the host is
  out, no answer may claim a confirmation newer than the last fetch that
  succeeded, and the named rewritten pages on the other hosts must still be
  reflected within the declared bound; after restoration, the host's pages
  must be reflected again within the declared bound without the ledger
  showing the host refetched from the start.
- A named page changes twice within one second, keeps its `Last-Modified`,
  and truthfully answers 304 to a conditional request. The changed content
  must still be reflected eventually, and the staleness statement must never
  treat that 304 as a confirmation of the bytes.
- The harness installs a redirect chain that makes two named URLs one page.
  The index must converge on one document for that page, and no query may
  return both identities.
- When a named page is acknowledged, the crawler is killed and restarted
  mid-crawl. The crawl resumes, the ledger shows no disallowed fetch and does
  not show the corpus refetched from the start, and the staleness statement
  remains honest across the gap.

Verification does not inspect private functions and does not look for a named
design. It reads the harness ledger and the answers of the public search API,
sampled throughout the run so that each named page's history in the index is
reconstructed from answers alone, and compares both against the declared
schedule of content changes.

## Acceptance evidence

A disallowed path never appears in the ledger as fetched, across the whole
run and every fault, and the ledger shows a single product token across the
run. For every named changed page, the sampled answers of the search API show
the old content replaced by the new within the declared bound, in fetch
order. Every staleness statement sampled during the run is no newer than the
fetch that confirmed the content behind it. The two redirected identities
resolve to one document. The restart leaves no double-indexed page and no
politeness violation. No request arrives before a wait the harness named has
passed, the withdrawn host's pages are reflected again within the declared
bound after its restoration, and the run ends with every host granting the
full ceiling and the crawler's cadence on every faulted host recovered.

The evidence report includes coverage of the corpus over time, the ratio of
304 to full responses on revisits and the bytes each carried, the
distribution of content age behind answered queries against the declared
bound, its derivation, and the counts that make it admissible — K, R, E, U,
and A(F) over the prepared overlap window — the division of each host's
allowance among content fetches, `robots.txt` fetches, and retries, and the
declared pagination contract holding while paginating past ten thousand
results during active indexing. It names the pages the budget never reached
and states what the product guarantees about them, names the eligible pages
that lost service to the work the declared bound promised, states the budget
the fault windows withheld — the full ceiling less the granted allowance, read
from the harness's allowance record — and the learner's own account of what
that cost the crawl, and it names one residual limitation.

## Neighbouring systems

A practitioner might have reached for one of these instead. The names and
their documentation links publish into `README.md`; the boundary difference
stated with each publishes into `HINTS.md`, because naming what a neighbour
does differently here points at this lab's quirk.

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

## Scope

The expected focused time is eighteen to twenty-two hours; extraction and
robots parsing are prepared, but strict RFC 9309 compliance under fault,
crash-safe crawl state, the revisit-versus-discover budget, and a ceiling
whose breach costs future budget still price out well above a phase 1-4 lab.
The learner builds the crawl,
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

Every citation below is solution-bearing. None of it publishes into
`README.md`; it belongs in `HINTS.md` or `EVALUATION.md`.

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../docs/search-and-retrieval-track.md)
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
  truthful 304 is not proof the bytes are unchanged. Solution-bearing: this
  belongs in `HINTS.md`, never in `README.md`.
- [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html) — HTTP caching:
  freshness lifetime and the heuristic of a fraction of the age since
  `Last-Modified` (§4.2.2), and updating a stored response on 304 (§4.3.4).
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [RFC 6585](https://www.rfc-editor.org/rfc/rfc6585.html) — additional HTTP
  status codes: 429 means the client has sent too many requests in a given
  amount of time, and the response may carry a `Retry-After` header saying how
  long to wait before making a new request.
- [Reduce Google's crawl rate](https://developers.google.com/search/docs/crawling-indexing/reduce-crawl-rate)
  — Google's crawling infrastructure reduces a site's crawl rate when it meets
  a significant number of 500, 503, or 429 responses, warns that sustaining
  them longer than one to two days can harm how the site appears in Google
  products, and raises the rate again automatically once the errors fall.
  Solution-bearing: this belongs in `HINTS.md`, never in `README.md`.
- [SEC EDGAR access policy](https://www.sec.gov/os/accessing-edgar-data) — a
  ceiling published as live policy: a current maximum request rate of ten
  requests per second, a declared user agent expected in request headers, and
  automated tools outside the acceptable policy not allowed to crawl the site.
- [OpenSearch refresh](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/)
  — an indexed document becomes searchable only after a refresh, which runs
  every second by default, so the index lags its own writes and that lag is
  part of any honest staleness statement. Solution-bearing: this belongs in
  `HINTS.md`, never in `README.md`.
- [OpenSearch pagination](https://docs.opensearch.org/latest/search-plugins/searching-data/paginate/)
  — result windows cap at ten thousand documents by default, and paginating
  deeper or consistently while the index changes requires the mechanisms this
  page names. Solution-bearing: this belongs in `HINTS.md`, never in
  `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
