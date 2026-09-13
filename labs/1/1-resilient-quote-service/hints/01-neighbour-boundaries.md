> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Envoy** — moves admission control, retry budgets, and outlier detection
  into a proxy in front of the service, so the overload policy lives in
  configuration and the application never learns its own capacity.
- **resilience4j** — with Hystrix before it, packages the failure policy as
  named per-call-site primitives, which fixes the isolation boundaries before
  any measurement has shown where the actual bottleneck sits.
- **HAProxy** — caps connections and request rates at the edge, so excess
  traffic is refused before it reaches the service rather than handled
  inside it.
