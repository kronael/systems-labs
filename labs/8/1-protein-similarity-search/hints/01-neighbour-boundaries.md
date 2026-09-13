> Spoilers. Open only when stuck.

# What the neighbours do differently

- **NCBI BLAST+** is the reference product as a batch command: the database
  is built once and frozen, and significance is computed against that
  snapshot, so corpus growth means rebuild-and-rerun rather than a serving
  contract with named corpus versions.
- **DIAMOND** aligns proteins at 100x to 10,000x the speed of BLAST by
  organizing work as batch analysis of big sequence data, which buys
  throughput for whole-dataset jobs and gives up the per-query serving path
  this lab is about.
- **OpenSearch k-NN vector search** replaces string similarity with distance
  between embeddings; approximate search "requires a sacrifice in accuracy
  but increases search processing speeds appreciably", which moves the
  corpus dependence out of the score and into which neighbours are found at
  all.
