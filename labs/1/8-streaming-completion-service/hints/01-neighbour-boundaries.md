> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Ray Serve** — scales replicas and queues requests in front of them, so its
  answer to pressure is another replica rather than a decision about work
  already in flight; the reclamation inside one replica's engine stays
  invisible to it.
- **Text Generation Inference** — ships the server and the engine as one
  product with its own queue and its own limits, so the admission policy is
  configuration rather than a design you own and defend.
- **NVIDIA Triton Inference Server** — batches across models and frameworks at
  the inference boundary, which makes the unit of work a batched inference
  rather than a long-lived stream whose cost grows while it runs.
