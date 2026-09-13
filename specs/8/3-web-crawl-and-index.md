---
status: draft
---

# Web crawl and index — failure schedule

This file holds the one part of the lab that must never reach the learner.
The task, the hints and the citations live in the lab directory.

## Adversarial evaluation

Faults fire at named barriers, never on a timer and never at random, so every
run produces the same history and verification asserts exact URL identities
against the harness ledger.

- When a named page is acknowledged as indexed, the harness rewrites its
  content. The change must be reflected in search results within the declared
  bound, and no answer in between may claim a confirmation newer than the last
  fetch that saw the old content.
- At a named fetch, a host's `robots.txt` begins returning server errors; at
  a later named fetch it recovers. While it is unreachable, the ledger must
  show no fetch on that host beyond what the standard's caching guidance
  permits, and the refetches those errors invite must stay inside the host's
  one allowance: the window must end with the full ceiling intact.
- When a named fetch on a named host is acknowledged, the harness cuts that
  host's allowance to half the rate the crawler itself held over a window the
  schedule fixes, and answers anything faster with 429 and a `Retry-After`
  naming the wait — the cut chases the crawler's own cadence, so every design
  that spends its budget meets it. The ledger must show at least one 429 on
  the host and, after each, no request before its named wait has passed; once
  a named count of requests has arrived on time the full ceiling is restored,
  the host's named rewritten pages must still be reflected within the
  declared bound, and the crawler's cadence on the host must return to what
  it held before the cut, within a precision the learner declares.
- While that allowance is cut, at a named fetch the same host's `robots.txt`
  begins returning server errors; at a later named fetch it recovers. The
  content fetches a cached copy still permits, the refetches the errors
  invite, and the retries the named waits schedule now contend inside half an
  allowance. The ledger must show the host's whole traffic inside it, no
  request before its named wait, and the host's named rewritten pages
  reflected within the declared bound after both recoveries.
- When a named page is acknowledged as indexed, the harness withdraws its host
  from the crawler's product token — every request under that identity, on
  any connection, is refused with 429 and a `Retry-After` naming the wait —
  and restores it once a named count of requests has landed on the other
  hosts; a refused request at the withdrawn host does not postpone the
  restoration, but its named wait binds like any other. While the host is
  out, no answer may claim a confirmation newer than the last fetch that
  succeeded, and the named rewritten pages on the other hosts must still be
  reflected within the declared bound; after restoration, the host's pages
  must be reflected again within the declared bound without the ledger
  showing the host refetched from the start.
- A named page changes twice within one second, keeps its `Last-Modified`,
  and truthfully answers 304 to a conditional request. The changed content
  must still be reflected eventually, and the staleness statement must never
  treat that 304 as a confirmation of the bytes.
- The harness installs a redirect chain that makes two named URLs one page.
  The index must converge on one document for that page, and no query may
  return both identities.
- When a named page is acknowledged, the crawler is killed and restarted
  mid-crawl. The crawl resumes, the ledger shows no disallowed fetch and does
  not show the corpus refetched from the start, and the staleness statement
  remains honest across the gap.

Verification does not inspect private functions and does not look for a named
design. It reads the harness ledger and the answers of the public search API,
sampled throughout the run so that each named page's history in the index is
reconstructed from answers alone, and compares both against the declared
schedule of content changes.
