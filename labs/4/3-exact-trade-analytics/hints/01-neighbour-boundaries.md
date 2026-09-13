> Spoilers. Open only when stuck.

# What the neighbours do differently

- **PostgreSQL** — enforces uniqueness in the write path, so the duplicate
  problem never reaches the reader — and pays for it with write cost and a
  table size this workload would not tolerate.
- **Apache Druid / Apache Pinot** — target the same interactive analytical
  queries but organize ingestion around segments and real-time versus
  historical nodes, which moves the freshness question into the topology.
- **Elasticsearch or OpenSearch** — would make the top-N and recent-window
  queries easy and the exactness guarantee harder, because scoring and
  refresh intervals sit between a write and its visibility.
