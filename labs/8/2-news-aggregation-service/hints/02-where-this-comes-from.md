> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../specs/01-systems-labs.md) — shared scaffold,
  submission, evidence, and verification contracts.
- [`../../docs/search-and-retrieval-track.md`](../../../../docs/search-and-retrieval-track.md)
  — the track record where this candidate is expanded.
- [Broder, *Identifying and Filtering Near-Duplicate Documents*](https://cs.brown.edu/courses/cs253/papers/nearduplicate.pdf)
  — resemblance is a number between 0 and 1 computed from a document's
  shingle set, so "roughly the same" is a continuous measure and any
  duplicate boundary is a chosen point on it. Solution-bearing: this belongs
  in `hints/`, never in `README.md`.
- [Manku, Jain, Das Sarma, *Detecting Near-Duplicates for Web Crawling*](https://research.google.com/pubs/archive/33026.pdf)
  — 64-bit fingerprints over billions of pages, where near-duplicates differ
  in small fragments such as advertisements, counters, and timestamps, and
  the distance parameter trades false positives against false negatives with
  no natural sharp value. Solution-bearing: this belongs in `hints/`,
  never in `README.md`.
- [RSS 2.0 specification](https://www.rssboard.org/rss-specification) — an
  item's `pubDate` and `guid` are both optional, guid uniqueness is left to
  the source of the feed, and an aggregator "may choose" to use the guid to
  decide whether an item is new; nothing guarantees timestamps are present,
  accurate, or stable.
- [RFC 4287, The Atom Syndication Format](https://www.rfc-editor.org/rfc/rfc4287)
  — `atom:id` MUST NOT change when an entry is relocated or republished and
  revisions retain the same id, while `atom:updated` changes only when the
  publisher considers a modification significant and date values need only
  be "as accurate as possible": content changes under a stable identifier
  without the timestamps confessing.
- [OpenSearch Refresh Index API](https://docs.opensearch.org/latest/api-reference/index-apis/refresh/)
  — a document is not searchable until a refresh converts in-memory
  structures into searchable segments, and refreshes run on an interval that
  idles without search traffic, so a write and its visibility are separate
  events.
- Implementation pointers do not exist while the spec is `draft`.
