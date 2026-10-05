> Spoilers. Open only when stuck.

# Where the behaviour in this lab is reported

Each page below reports the real behaviour this lab rests on. Reading one
answers part of the design, which is why they are here and not in the task.

- [vLLM optimization and tuning](https://docs.vllm.ai/en/latest/configuration/optimization.html)
  — the engine logs that a "Sequence group 0 is preempted by
  PreemptionMode.RECOMPUTE mode because there is not enough KV cache space",
  and preempted requests "are recomputed when sufficient KV cache space
  becomes available again". Work the engine already did is thrown away and
  done again, and the request was accepted before any of that.
- [vLLM automatic prefix caching](https://docs.vllm.ai/en/latest/design/prefix_caching.html)
  — cached blocks are evicted from a fixed pool in least-recently-used order,
  so what survives depends on what else is running.
- [vLLM CPU installation](https://docs.vllm.ai/en/latest/getting_started/installation/cpu.html)
  — how the engine runs without a GPU, which is how this lab runs it.
