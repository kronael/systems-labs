> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../../../docs/search-and-retrieval-track.md)
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
  belongs in `hints/`, never in `README.md`.
- [RFC 9111](https://www.rfc-editor.org/rfc/rfc9111.html) — HTTP caching:
  freshness lifetime and the heuristic of a fraction of the age since
  `Last-Modified` (§4.2.2), and updating a stored response on 304 (§4.3.4).
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [RFC 6585](https://www.rfc-editor.org/rfc/rfc6585.html) — additional HTTP
  status codes: 429 means the client has sent too many requests in a given
  amount of time, and the response may carry a `Retry-After` header saying how
  long to wait before making a new request.
- [Reduce Google's crawl rate](https://developers.google.com/search/docs/crawling-indexing/reduce-crawl-rate)
  — Google's crawling infrastructure reduces a site's crawl rate when it meets
  a significant number of 500, 503, or 429 responses, warns that sustaining
  them longer than one to two days can harm how the site appears in Google
  products, and raises the rate again automatically once the errors fall.
  Solution-bearing: this belongs in `hints/`, never in `README.md`.
- [SEC EDGAR access policy](https://www.sec.gov/os/accessing-edgar-data) — a
  ceiling published as live policy: a current maximum request rate of ten
  requests per second, a declared user agent expected in request headers, and
  automated tools outside the acceptable policy not allowed to crawl the site.
- [OpenSearch refresh](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/)
  — an indexed document becomes searchable only after a refresh, which runs
  every second by default, so the index lags its own writes and that lag is
  part of any honest staleness statement. Solution-bearing: this belongs in
  `hints/`, never in `README.md`.
- [OpenSearch pagination](https://docs.opensearch.org/latest/search-plugins/searching-data/paginate/)
  — result windows cap at ten thousand documents by default, and paginating
  deeper or consistently while the index changes requires the mechanisms this
  page names. Solution-bearing: this belongs in `hints/`, never in
  `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
