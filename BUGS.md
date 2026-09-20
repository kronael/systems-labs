# Bugs

The queue is empty.

Record a defect here rather than fixing it in passing — see `CONTRIBUTING.md`,
"Filing a defect". A fix that changes a contract, control flow, or anything
cross-cutting is a redesign: record it as a proposal and get sign-off first.

## S39 — a lab on object storage has no hosted comparison (2026-09-20, proposed)

A candidate lab publishes versioned dataset releases to S3-compatible object
storage, run locally as SeaweedFS. Its gate proves a declared model, and
`docs/contract.md` requires that such a gate be compared against the hosted
service by `make smoke`. S3 is not on the contract's optional-cloud list,
which names Lambda, SQS, DynamoDB on-demand and short logs.

Either the contract gains S3 on that list, or the lab ships with no hosted
comparison and says so. The cost of the comparison is real: at the advertised
$0.023 per GB-month of storage and $0.09 per GB transferred out, one release
of the size the lab's domain implies is roughly ten dollars a run.

This is a contract change and needs sign-off before any lab text exists.

- **Severity:** medium — it blocks one candidate lab, nothing shipped
- **Scope:** `docs/contract.md`, optional-cloud list
- **Affected:** the proposed object-storage lab
- **Source:** review of the object-storage proposal, 2026-09-20
- **Status:** proposed
- **Fix:**
