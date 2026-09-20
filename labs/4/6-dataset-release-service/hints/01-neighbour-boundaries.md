> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Apache Iceberg** — makes a table version current by swapping one pointer
  under a catalog that provides the atomic compare-and-set the object store
  does not, so the hard part is delegated to a component this lab does not
  have.
- **lakeFS** — puts a versioned namespace in front of the store and gives you
  branches and commits, which turns publication into a merge; the partially
  written objects still exist underneath, but nothing in your design has to
  know about them.
- **DVC** — versions data by content hash beside the code that produced it,
  so "current" lives in a git commit rather than in the store, and readers
  come to it instead of polling.
