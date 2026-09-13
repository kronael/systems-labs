# Bugs

The queue is empty.

Record a defect here rather than fixing it in passing — see `CONTRIBUTING.md`,
"Filing a defect". A fix that changes a contract, control flow, or anything
cross-cutting is a redesign: record it as a proposal and get sign-off first.

## S33 — six phase 1 labs cannot be worked by predict, provoke, observe (2026-09-13, open)

A review of every phase 1 spec against the learner method — predict the failure
mode, provoke it, watch it manifest — found one lab clean and six with a gap.
Each entry names the smallest change that closes it.

- `1/5:76-78` names five barriers and `1/5:82-90` declares adapters for four.
  "Its local publication progress" has none, and the same paragraph says a
  barrier the run never maps fails the run, so a correct design fails by
  construction. The commit and broker-acknowledgement barriers already bracket
  that point.
- `1/1:87` raises one provider's latency at a named request but never ties the
  delay to the validity left on the quote that request returns. A naive merge
  can pass without ever returning an expired quote, so the lab's headline
  prediction has nothing to falsify it.
- `1/4:172` fires its kill at "a named item identity has been published in the
  new shape and its neighbour has not". "Neighbour" is undefined and no public
  adapter exists for it. The lab also names no fault layer, which the fault
  injection contract requires.
- `1/3:104` asks that accepted activity produce "the correct current and
  aggregate views" and names no independent oracle; `1/3:109` compares a
  rebuild against the learner's own output. `1/6` and `1/7` recompute from
  retained records instead.
- `1/7:293` provokes the gap only past the learner's declared reaction
  allowance, and `1/7:220-222` gives that allowance no ceiling. A learner who
  declares an allowance longer than the run never sees the violation.
- Scope sections hand over the prediction in `1/2:199-201`, `1/4:240`, and
  `1/6:212-215`. `1/7` marks the equivalent passage `HINTS.md`-bound.

- **Severity:** high — the failure gate is the lab, and in `1/5` a correct
  design fails the run
- **Scope:** phase 1 specs
- **Affected:** `specs/1/1`, `1/2`, `1/3`, `1/4`, `1/5`, `1/6`, `1/7`
- **Source:** review of specs/1/ against the learner method, 2026-09-13; the
  `1/5` and `1/1` findings re-verified against the file text
- **Status:** open
- **Fix:**

## S34 — four phase 2 labs lack the supplier or the barrier their own gate needs (2026-09-13, open)

The same review against phase 2, where the environment hides its limits and the
controller has to supply them. `2/2` and `2/3` support the method; the rest have
gaps, and one is shared.

- **Shared.** `2/5:44` names the controller as the thing that "throttles above
  the declared ceiling". `2/1:49`, `2/3:156-157` and `2/4:130-131` all demand a
  rejected fraction under load, and `01:604-606` says the emulators do not model
  throttling, so in three labs nothing local supplies it. The `2/5` sentence,
  once per lab, closes it.
- `2/5:78-80` tells the learner that "an expiry judged from anything the process
  itself recorded is not evidence", which rules out the naive design before the
  learner commits to it. It belongs in `HINTS.md`. The lab also schedules no
  thaw held past a named quote's expiry, so its own subject never fires.
- `2/4:105` and `2/4:161-162` ask for contention the controller's transport
  layer recorded at hot accounts; the scaffold sets up no transport layer, and
  DynamoDB Local applies no per-key limit. `2/4:67-68` requires correctness
  under every arrival order while the schedule never reorders. `2/4:127` places
  a freeze "between the ledger write and the audit publication", a point a
  design that publishes before responding does not have.
- `2/1:92-93` forbids one account's data leaking into another's response with
  no barrier that forces the reuse, and `2/1:132-133` names a two-request
  logical operation the public API does not have.
- `2/3:57-58` has the controller record "the resource key that request
  addresses" while the store's data layout is the learner's, and nothing says
  how the controller finds the resource inside a request.
- `2/2` names no fault layer, which the fault injection contract requires.

The review also flagged `HOWTO.md` for promising a fault the learner can move.
Fixed in `bdd3f83` before this entry was written.

- **Severity:** high — three labs demand evidence the environment cannot produce
- **Scope:** phase 2 specs
- **Affected:** `specs/2/1`, `2/2`, `2/3`, `2/4`, `2/5`
- **Source:** review of specs/2/ against the learner method, 2026-09-13; the
  `2/5` leak and the throttling gap re-verified against the file text
- **Status:** open
- **Fix:**

## S35 — five separate-catalog labs cannot provoke the failure they name (2026-09-13, open)

The same method review across phases 6, 7 and 8. Eight of the sixteen labs are
clean: `6/1`, `6/3`, `6/4`, `6/5`, `7/2`, `7/4`, `7/6`, `8/2`.

- `8/1:44` starts one OpenSearch, `8/1:217-218` puts multi-node operation
  outside the problem, and `8/1:143-146` restarts "one OpenSearch node" and
  forbids "a quietly smaller answer assembled from the shards that survived".
  On one node there are no surviving shards, so the partial answer the lab is
  about cannot happen. Issue the named query while the restarted node's shards
  recover, or fail one named shard.
- `7/1:126-127` freezes the validator "long enough that slots in the frozen
  window are skipped when it resumes". The `SlotStatus` citation supplies the
  vocabulary of skipped and dead slots; no cited page reports that a freeze
  produces them. Cite the causation or provoke the invariant with the
  kill-and-restart case that issue 27842 already grounds.
- `6/2:81-82` returns one I/O error to a flush and then resumes, and only
  `6/2:93-94` checks that the error reached the caller. No data is dropped
  behind the failed flush, so a design that retries into a false success still
  serves every record.
- `7/7:172-173` keys a barrier on the service "having written that
  authorization off", a state the requirements never demand, so a design that
  waits forever never reaches it and the run fails a correct design.
- `8/5:59-60` asks which queries influenced a report while `8/5:36-37` keeps
  candidates' true effects hidden from the harness, so the held-out check
  cannot be provoked on the prepared overfit candidate.
- Smaller: `7/3:113-114` never forks across the indexer's downtime, so the
  withdrawal it warns about at `7/3:93-94` never fires; `8/3:214-215` and
  `8/3:48` disagree about whether a page's `ETag` changes, which decides
  whether the fault fires at all; `8/4:72-73` hands the learner the naive
  shape instead of asking for the comparison.

- **Severity:** medium — no correct design fails a run except in `7/7`, but
  four labs cannot show the failure they are built around
- **Scope:** phase 6, 7 and 8 specs
- **Affected:** `specs/8/1`, `7/1`, `6/2`, `7/7`, `8/5`, `7/3`, `8/3`, `8/4`
- **Source:** review of specs/6, /7, /8 against the learner method, 2026-09-13;
  the `8/1` contradiction and the `7/1` citation gap re-verified against the
  file text
- **Status:** open
- **Fix:**

## S36 — `4/1` cannot provoke its own quirk, and `3/2` fires "around" a barrier (2026-09-13, open)

The method review across phases 3 and 4. `3/1`, `4/2` and `4/3` support the
method as written.

- **`4/1` is the worst finding of the four reviews.** Its requirements never say
  the store refuses work: `4/1:51-53` gives only the hot-symbol skew, and the
  capacity ceiling appears solely in `Adversarial evaluation`, which never
  publishes, so the learner cannot predict the failure at all. DynamoDB Local
  then cannot produce it either — its own usage notes say read and write speed
  "is limited only by the speed of your computer", and that reads, though
  eventually consistent, mostly appear strongly consistent, which also kills
  the `4/1:78-79` read-after-write fault. Requirements need a declared capacity
  budget with its own refusal outcome, and the transport model needs index lag
  or the read-after-write fault goes.
- **`3/2:104` kills a task "around a PostgreSQL effect and checkpoint".**
  `01:357-358` says an atomic effect admits no interior barrier and a fault
  lands immediately before or immediately after it, never around it. `3/2:105`
  restarts workers "during rebalance", a phase rather than a barrier, and the
  section is missing the "Every fault fires at a named barrier" sentence its
  neighbours carry. The learner cannot name the state that proves the
  prediction, because the side is undecided.
- **`4/5` cites a Lambda freeze it never provokes.** The citation at
  `4/5:158-161` describes buffered state surviving only if the same environment
  thaws; the requirements name only batch, visibility, repetition and
  concurrency, and the schedule suspends nothing. The lab also names no layer
  for its gate. Either cut the freeze citation or add one process-layer fault.
- Editorial: `4/2:80-81` repeats `4/2:75-76` with a `HINTS.md`-bound mark the
  first copy lacks.

Shared with `S33` to `S35`: `01:837` says `make fault` takes "never an argument
the caller types", so nothing lets a learner choose the record or the hold. The
fix belongs in `specs/0/5-shared-scaffold.md`, not in the labs.

- **Severity:** high — `4/1` cannot be worked as written
- **Scope:** phase 3 and 4 specs
- **Affected:** `specs/4/1`, `3/2`, `4/5`, `4/2`
- **Source:** review of specs/3 and specs/4 against the learner method,
  2026-09-13; the `4/1` publication gap and the `3/2` "around" violation
  re-verified against the file text
- **Status:** open
- **Fix:**
