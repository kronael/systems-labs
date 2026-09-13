> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Knative** — runs event-driven, scale-to-zero workloads on Kubernetes
  itself, so both execution shapes share one platform and the lifecycle
  split this lab studies never appears — at the price of operating the
  machinery that hides it.
- **Temporal** — moves redelivery, retries, and progress into a workflow
  engine's durable execution, so acknowledgement stops depending on the
  transport and the platform difference hides behind a second stateful
  system.
- **Pulumi / CDK** — describe infrastructure in a general-purpose language
  rather than declarative HCL, so the set of resources is computed while the
  program runs rather than declared before it does. This lab checks drift
  and rollout, which rest on a plan that states every change in advance, and
  a general-purpose language would also put infrastructure in the same
  languages as the application it must stay separable from.
