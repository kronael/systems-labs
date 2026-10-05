> Spoilers. Open only when stuck.

# Hidden is not gone

Read the store's own pages on deletion before you decide what your completion
statement can claim. Three facts decide it.

- A lightweight delete marks rows rather than removing them. "The mutation
  does not physically delete the rows that have been marked as deleted, this
  will only happen during the next merge", so "for an unspecified period,
  data is not actually deleted from storage and is only marked as deleted".
- A query can be told to read the marked rows anyway, which is how a check
  tells hidden from gone rather than taking the service's word for it.
- For a non-replicated table, the store's own record of when a mutation
  finished is "tracked in memory and is reset when the table is reloaded (e.g.
  on server restart)", so the answer to "when did it finish" disappears exactly
  when you most want it.

A row-versioning engine reaches the same place by another road: a delete is a
row like any other, and the last row for a key is kept even when it is the
delete, unless a cleanup is asked for explicitly.

- <https://clickhouse.com/docs/sql-reference/statements/delete>
- <https://clickhouse.com/docs/operations/system-tables/mutations>
- <https://clickhouse.com/docs/engines/table-engines/mergetree-family/replacingmergetree>
