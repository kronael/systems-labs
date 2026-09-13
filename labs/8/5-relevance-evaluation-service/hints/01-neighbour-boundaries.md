> Spoilers. Open only when stuck.

# What the neighbours do differently

- **trec_eval** — is the TREC community's standard scorer: it judges a
  frozen run file against a frozen judgement file, so the engine, the
  corpus, and serving sit entirely outside its world — nothing can drift
  between two invocations, and nothing serves traffic either.
- **Quepid** — puts collaborative judgement collection first and evaluates
  its cases against the live engine, so judgements evolve continuously and a
  case's score follows whatever the index holds now rather than a pinned
  corpus version.
- **Elasticsearch's ranking evaluation endpoint** — makes evaluation a
  stateless engine call: it runs the supplied rated queries against the live
  index and returns a metric, keeping no run history, so identity and
  comparability across runs are entirely the caller's problem.
