---
status: draft
---

# Portable market ingestion

## Brief

Design and build a market-record normalization system that runs in two prepared
environments: a long-lived Kubernetes deployment consuming Kafka and a Lambda-
compatible batch deployment consuming an SQS-compatible queue. Both expose the
same accepted record and query contract, but the architecture must account for
their different lifecycle, delivery, scaling, and cost behavior.

CloudEvents is the required input envelope, DynamoDB is the required result
API, Kubernetes and Lambda are the required compute models, and OpenTofu is the
required infrastructure tool. The prompt does not prescribe shared code
boundaries, adapters, deployment units, queue handling, rollout policy,
resource graph, module layout, or configuration mapping.

## Prepared scaffold

The local scaffold has three prepared layers:

- a dependency-only Compose profile with Kafka, ElasticMQ, DynamoDB Local,
  OpenTelemetry collection, generators, and the fault controller;
- a `kind` cluster with registry access, ingress-free service discovery,
  telemetry wiring, and an empty learner workload slot;
- a Lambda-compatible batch runner with SQS events and DynamoDB Local.

OpenTofu contains tested sandbox modules for the local Kubernetes control plane
and an optional bounded AWS account shell. The learner owns the application
topology and extends the resource graph for it. Standard Make targets create
each sandbox, validate the untouched scaffold, deploy the learner system,
inject rollout and delivery faults, show drift, run the shared contract suite,
and collect evidence.

## Requirements

Both environments accept the same generated market CloudEvents and expose the
same normalized values, validation outcomes, stable source identities,
query behavior, and semantic telemetry fields. Duplicate delivery cannot
corrupt results. Rejected and retryable records remain distinguishable.

The Kubernetes system survives worker replacement, rebalance, SIGTERM, and a
faulty rollout without losing acknowledged work. The Lambda-shaped system
handles mixed-success batches, visibility expiry, repeated invocation, and
bounded concurrency. Platform-specific state must remain outside the shared
domain contract.

The scale target is 20 million market events through each environment, a
sustained ingest of 5,000 events per second on the long-lived path, and a
queue backlog drained in bounded batches under an invocation concurrency cap
of 10, deep enough that no single invocation can finish it. These numbers
size the problem, not the pass bar: throughput and recovery are measured
against the learner's declared service level, not a fixed number.

OpenTofu owns every submitted infrastructure resource. An out-of-band change
must appear in the next plan. Secrets may not appear in configuration output,
plan text, or state. The local run proves all required gates; AWS is optional.

## Architecture questions

The submitted `ARCHITECTURE.md` must explain:

- which behavior is genuinely shared and which belongs to each platform;
- how acknowledgement and recovery differ between Kafka and SQS delivery;
- how lifecycle and graceful termination work for each compute model;
- how configuration, identity, and secrets enter each environment;
- how one result identity tolerates redelivery from either transport;
- how deployment, readiness, rollback, and unfinished work interact;
- how OpenTofu modules and state reflect ownership and dependency boundaries;
- which workload fits each platform and where portability ends;
- how the cost comparison is derived from measured local or optional cloud
  evidence.

At least two application-decomposition or deployment designs must be compared.

## Adversarial evaluation

The grader sends identical event histories through both environments, kills a
Kubernetes worker around a record boundary, sends SIGTERM under load, deploys a
version that fails readiness, rolls back with backlog present, expires an SQS
lease, fails one batch item, repeats invocations, changes a managed Kubernetes
field outside OpenTofu, and scans plan and state for supplied secret sentinels.

Evaluation uses public input and query contracts, transport-visible histories,
DynamoDB results, Kubernetes rollout state, Lambda-shaped responses, OpenTofu
plans and state, traces, resource use, and cost evidence. No fixed application
process count or adapter pattern is required.

## Acceptance evidence

The two environments produce identical domain results for the same accepted
source identities. Faulty rollout and worker replacement preserve the stated
acknowledgement contract. Repeated delivery has one logical result. Drift is
reported, secrets remain absent from state, and recovery behavior is visible
to users and operators.

The report compares end-to-result latency, startup behavior, backlog age,
throughput, resource bounds, duplicate attempts, rollout and recovery time,
infrastructure changes, and cost. It identifies behaviors the local Lambda
runner and `kind` cluster cannot prove about AWS or managed Kubernetes.

## Neighbouring systems

A practitioner might have reached for one of these instead. Each changes the
boundary this lab is about, and each is worth reading about before defending
the design:

- **Knative** runs event-driven, scale-to-zero workloads on Kubernetes
  itself, so both execution shapes share one platform and the lifecycle split
  this lab studies never appears — at the price of operating the machinery
  that hides it.
- **Temporal** moves redelivery, retries, and progress into a workflow
  engine's durable execution, so acknowledgement stops depending on the
  transport and the platform difference hides behind a second stateful
  system.
- **Pulumi** and **CDK** describe infrastructure in a general-purpose
  language rather than declarative HCL, so the set of resources is computed
  while the program runs rather than declared before it does. This lab grades
  drift and rollout, which rest on a plan that states every change in advance,
  and a general-purpose language would also put infrastructure in the same
  languages as the application it must stay separable from.

Read their documentation on lifecycle, delivery, and state. The lab does not
run them.

## Scope and cost

The expected focused time is six to eight hours. Compose dependencies, `kind`,
the Lambda-compatible runner, base OpenTofu sandboxes, telemetry, workload, and
faults are prepared. MSK, EKS, NAT gateways, RDS, ElastiCache, API Gateway,
multi-region deployment, and mandatory AWS access are outside the problem.

## Code pointers

- [`../01-systems-labs.md`](../01-systems-labs.md) — shared scaffold,
  portability, cost, IaC, platform, and evidence contracts.
- [`../0/1-lab-selection.md`](../0/1-lab-selection.md) — selection rationale.
- [Execution environment lifecycle](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html)
  — Lambda freezes the environment between invocations and recycles it within
  hours, so unfinished background work and buffered state survive only if the
  same environment happens to thaw, and `/tmp` outlives a freeze but not a
  replacement.
- [Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
  — an invocation runs at most 900 seconds, `/tmp` offers 512 MB to 10 GB,
  and concurrent executions default to 1,000 per account per Region.
- Implementation pointers do not exist while the spec is `draft`.
