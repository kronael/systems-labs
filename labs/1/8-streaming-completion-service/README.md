# Streaming completion service

Design and build a service that accepts generation requests and streams each
answer back token by token. Every accepted request carries two promises: the
caller sees the first token within a stated bound, and the answer completes
within a second, longer bound. The service runs one generation engine over a
memory pool it cannot grow, and the engine serves many requests at once out of
that one pool.

The assignment is the whole service: the public streaming interface, admission,
the promise accounting, the telemetry, and end-to-end tests. How work is
admitted, how the promises are computed and defended, what the caller is told
when a promise cannot be kept, and how the service divides into processes are
the learner's decisions.

## What you are given

The supplied Compose stack starts one vLLM server on CPU with a small
instruction model and a fixed pool size, OpenTelemetry collection, a metrics
backend, and the fault controller. The course also supplies generated request
workloads with declared prompt and answer length distributions, an open-loop
load generator, HDR histogram output, and the fault schedules behind
`make fault`.

The learner owns the service in front of the engine and its Compose layer. The
common Makefile starts dependencies, validates scaffold health, runs tests,
injects faults, runs load, and collects evidence. No cloud account and no GPU
are required.

## Requirements

The service accepts a generation request and streams its answer. Every
accepted request has a stable identity and names the caller that sent it, and
every identity reaches exactly one reported outcome: completed with its
answer, or ended early with a stated reason. An outcome, once reported, is
final.

Bound attainment is reported per caller, and the design states what one
caller's requests may cost the promises already made to other callers.

The two promises are the service's own, declared in its configuration and
reported in its evidence: a first-token bound and a completion bound. The
design states what the caller observes when the service can no longer keep a
promise it has already made, and that observation is part of the public
interface.

The engine's memory pool is fixed and shared by everything the engine is
working on at once. Whether a request is admitted, and what the service
promises when it admits one, are decisions the design owns; the pool's size is
an environment fact, and the design states what it assumes about it.

Answers are reproducible under the declared decoding settings: the same
request against the same model and settings yields the same answer, so a check
can compare an answer against an independently produced one by identity.

The scale target is 200 concurrent streaming callers, a sustained 40 accepted
requests per second, prompts between 200 and 6,000 tokens with one in twenty
above 4,000, and 100,000 requests per evidence run. The long prompts are the
shape of the domain: a support tool summarising a whole ticket thread sends
the thread, and the threads that matter most are the longest. Latency is
judged against the learner's declared bounds and the calibrated local
capacity, not a fixed number.

## What your ARCHITECTURE.md must explain

The submitted `ARCHITECTURE.md` must explain:

- what the service knows about the engine's remaining capacity at the moment
  it decides to accept a request, and how it knows it;
- what a promise made at admission rests on, and what can change after
  admission;
- how the design tells a caller that a promise it already made will not be
  kept, and why that is better than the alternatives it considered;
- what the service does with a request whose work has already been paid for
  once;
- what the admission decision knows about the caller, and what one caller's
  burst may cost another caller's promise;
- which properties of the engine the design relies on, and which it
  explicitly does not.

At least two admission designs must be compared. The chosen design needs
stated failure modes and one residual limitation.

## Acceptance evidence

Every accepted identity reaches exactly one reported outcome, and no reported
outcome changes afterwards. Answers match the independently produced ones by
identity under the declared settings. The declared bounds are reported with
the fraction of accepted requests that met them, and the report defends both
bounds against the calibrated capacity rather than against a fixed number.

The evidence report includes offered and accepted rates, first-token and
completion distributions per caller, the observed occupancy of the engine's
pool, the work the engine repeated, and the fraction of accepted requests that
ended early with a stated reason. It explains the relationship between the
accepted rate and the two bounds, and ties the admission decision to the
measurements.

A design that accepts almost nothing has not met the scale target, and the
evidence must make that visible.

## What a practitioner might have used instead

A practitioner might have reached for one of these instead. The lab does not
run them. What each does differently at this lab's boundary is in `hints/`,
because saying it here would point straight at the answer.

- **Ray Serve** — [documentation](https://docs.ray.io/en/latest/serve/index.html)
- **Text Generation Inference** — [documentation](https://huggingface.co/docs/text-generation-inference/index)
- **NVIDIA Triton Inference Server** — [documentation](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/index.html)

## What is outside the problem

The expected focused time is fourteen to eighteen hours. The learner builds
the service in front of the engine and its tests. The engine, the model, the
workloads, telemetry, and fault injection are prepared. Model quality, prompt
design, fine-tuning, tokeniser internals, multi-model routing, GPU
scheduling, and any user interface are outside the problem.

Stuck? See `hints/`.
