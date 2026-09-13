> Spoilers. Open only when stuck.

# What the neighbours do differently

- **Google Cloud Run** — a long-lived process behind an autoscaler keeps the
  connection pool and the in-process cache while making capacity a scaling
  policy, which trades first-invocation latency for capacity held during idle
  periods.
- **Amazon API Gateway caching** — an API gateway with request-level caching
  answers repeat requests before any code runs, which removes the fan-out
  entirely for a hit and moves the freshness decision into configuration.
- **Istio** — a service mesh sidecar provides timeouts, retries, and circuit
  breaking outside the application, which is exactly the policy this lab
  requires the design to own and state.
