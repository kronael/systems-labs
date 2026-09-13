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
