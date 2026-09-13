> Spoilers. Open only when stuck.

# What the neighbour does differently

- **pgvector** — moves similarity into a vector space with exact or
  approximate nearest-neighbour search; the threshold does not disappear, it
  becomes a distance cutoff, and an approximate index changes which
  neighbours are returned at all.
- **PostgreSQL full-text search** — keeps retrieval inside the transactional
  store, so a committed write is searchable immediately under ordinary MVCC
  visibility and the acceptance-to-visibility gap disappears — paid for with a
  ranking model far below a dedicated engine's.
- **Apache Solr** — shares the same Lucene core, and its collapse and expand
  feature groups results by a single-valued field that must already exist on
  the document — query-time grouping that presupposes the clustering decision
  was already made at write time.
