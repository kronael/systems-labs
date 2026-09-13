> Spoilers. Open only when stuck.

# What the neighbour does differently

- **Common Crawl** — removes the crawl entirely: a nonprofit's shared corpus
  of hundreds of billions of pages, grown by billions of pages a month,
  consumed as archives instead of fetched. Freshness becomes whatever the
  shared crawler's schedule delivered, and no one consumer chooses what is
  revisited.
- **Apache Nutch** — ships the entire crawl loop — pending-URL state, fetch
  scheduling, parsing, and indexing into Solr or Elasticsearch — as a
  configurable batch pipeline on Hadoop data structures, so the revisit policy
  this lab makes the learner design is a plugin selected in configuration.
- **Apache StormCrawler** — treats the crawl as an unbounded low-latency
  stream on Apache Storm rather than a scheduled batch, keeping URL status in
  an external store such as OpenSearch, which moves the revisit decision into
  stream re-emission.
