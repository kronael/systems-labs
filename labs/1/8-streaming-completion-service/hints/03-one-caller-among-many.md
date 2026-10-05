> Spoilers. Open only when stuck.

# One caller among many

The engine does not know your callers. Read its scheduler documentation for
two facts and then decide what to do about them.

- The default policy is first come, first served: requests are handled in
  order of arrival. Nothing in the engine gives one caller less of the pool
  than another.
- Each iteration has a token budget shared by everything in flight, so a
  caller sending long prompts takes a share of every step, not only a share
  of the pool.

- <https://docs.vllm.ai/en/latest/api/vllm/config/scheduler/> — the scheduling
  policy and the per-iteration token budget.
- <https://docs.vllm.ai/en/latest/configuration/optimization.html> — how
  prefills and decodes share that budget.
