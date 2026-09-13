> Spoilers. Open only when stuck.

# What the neighbours do differently

- **MongoDB** — records in one collection are not required to carry the same
  fields and a field's type may differ between them, so a shape change is
  never an operation against the store at all: it moves entirely into the
  application, which gains no moment at which the whole collection is known
  to have changed.
- **MySQL with InnoDB** — the statement declares the interruption it is
  willing to cause and the server refuses the change outright when it cannot
  honour that declaration, so the operator learns before the change runs what
  this lab makes the design discover while serving traffic.
- **CockroachDB** — runs the change as a background job that holds no locks
  on the table data and rolls out the new shape while the previous one is
  still in use, which moves the coexistence of two shapes out of the
  application and into the database's own machinery — and turns the change
  into a job whose duration the operator does not bound.
