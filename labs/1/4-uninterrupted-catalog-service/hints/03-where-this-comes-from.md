> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [`../01-systems-labs.md`](../../../../docs/contract.md) — shared scaffold,
  PostgreSQL, evidence, and verification contracts.
- [Directive 98/6/EC](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:31998L0006)
  — Article 3: "The selling price and the unit price shall be indicated for
  all products referred to in Article 1, the indication of the unit price
  being subject to the provisions of Article 5", and "The unit price need not
  be indicated if it is identical to the sales price"; Article 5 lets Member
  States waive the indication where it "would not be useful because of the
  products' nature or purpose or would be liable to create confusion".
  Neutral: it explains why the change exists and why the new value is
  conditional, never how to adopt one, so it may publish in `README.md`.
- [PostgreSQL `ALTER TABLE`](https://www.postgresql.org/docs/current/sql-altertable.html)
  — the lock level differs per subform and "An `ACCESS EXCLUSIVE` lock is
  acquired unless explicitly noted", while "Adding a column with a volatile
  `DEFAULT` (e.g., `clock_timestamp()`), a stored generated column, an
  identity column, or a column with a domain data type that has constraints
  will cause the entire table and its indexes to be rewritten."
  Solution-bearing: the same page lists which subforms take a weaker lock, so
  it belongs in `hints/`, never in `README.md`.
- [PostgreSQL explicit locking](https://www.postgresql.org/docs/current/explicit-locking.html)
  — the mode taken by a change "Conflicts with locks of all modes", including
  the mode every plain read acquires, and "a transaction seeking either a
  table-level or row-level lock will wait indefinitely for conflicting locks
  to be released". Solution-bearing: this is why a waiting change stops
  arriving readers, so it belongs in `hints/`, never in `README.md`.
- Implementation pointers do not exist while the spec is `draft`.
