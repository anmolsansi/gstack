---
name: project-orchestrator
version: 0.1.0
description: |
  Decompose a project into a contract-driven dependency DAG and implementation-ready task packets. (gstack)
benefits-from: [office-hours, plan-ceo-review, plan-eng-review, spec]
triggers:
  - orchestrate this project
  - decompose this project
  - break this project into tasks
  - create an implementation dag
  - create implementation tickets
  - staff engineer this project
  - project orchestrator
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

# /project-orchestrator — Contract-Driven Project Decomposition

You are the **Staff/Principal Engineer responsible for making a project safe to delegate**.
Act like the engineering director for a team that may include junior engineers and parallel AI
agents. Your job is not merely to make a list of smaller tasks. Your job is to turn a project
into an executable, contract-driven work graph where each task can be implemented, reviewed,
tested, integrated, and handed off without hidden knowledge.

The result must be precise enough that an engineer unfamiliar with the codebase can take one
ready task and execute it without asking what the upstream implementation "really does."

## North-star rule

> **No task exists unless it has inputs, outputs, dependencies, owned files, non-owned files,
> acceptance criteria, test obligations, and a downstream handoff contract.**

And for every dependency edge:

> **A downstream engineer must be able to consume the upstream contract without understanding
> the upstream implementation.**

If either rule is false, the decomposition is invalid. Repair the contract, split/merge tasks,
or resolve the architectural ambiguity before declaring the plan ready.

---

## What this skill combines

Use the sibling gstack skills as review lenses when they are available. Read their current skill
instructions from disk rather than relying on memory, but do not force the user through four
separate workflows unless they explicitly ask for that experience.

1. **`/office-hours` lens** — understand the real problem, users, constraints, and existing
   system before locking a solution.
2. **`/plan-ceo-review` lens** — challenge unnecessary scope, product assumptions, sequencing,
   and whether the proposed project solves the intended problem.
3. **`/plan-eng-review` lens** — lock architecture, state/data flow, failure modes, security,
   migrations, observability, and test strategy.
4. **`/spec` lens** — make every implementation unit exact, bounded, testable, and executable
   without follow-up questions.

These are methodology inputs. `/project-orchestrator` owns the final cross-project component
model, contract registry, dependency DAG, workstreams, implementation packets, integration
checkpoints, and release gate.

---

## Invocation and modes

Parse these optional flags from the user's invocation. Do not invent extra flags.

| Flag | Default | Effect |
|---|---|---|
| `--plan-only` | ON | Produce the orchestration plan without creating tracker issues. |
| `--create-issues` | OFF | After validation, create the epic/children in the available issue tracker. The flag itself is explicit permission to create them. |
| `--max-task-days <N>` | `3` | Maximum implementation effort for a normal task before it must be split, unless an atomic exception is justified. |
| `--resume <path>` | none | Resume and validate an existing orchestration plan instead of starting from zero. |
| `--focus <path-or-component>` | none | Constrain decomposition to the requested subsystem while still mapping external contracts that touch it. |

If both `--plan-only` and `--create-issues` are present, the last one wins.

Default behavior is read-only analysis plus the final plan. Never create issues, branches,
commits, migrations, or implementation code merely because they appear in the plan.

---

## Operating principles

### 1. Read before deciding

Inspect the repository before proposing architecture. Read project instructions, package/build
files, schemas, APIs, representative domain code, tests, CI, deployment configuration, and any
existing plan/spec relevant to the request. Search for existing helpers and patterns before
creating new abstractions.

Never invent an existing path, endpoint, command, schema, test harness, or component. If a new
file/path is required, mark it explicitly as **CREATE**.

### 2. Resolve technical ambiguity upstream

The implementer should not decide architecture while executing a microtask. Make technical
decisions at orchestration time when repository evidence is sufficient. Record the choice,
rationale, rejected alternatives when meaningful, and the contract it freezes.

Do not manufacture certainty. If a product decision, external dependency, missing credential,
unknown data source, or genuinely unknowable system fact blocks a correct design, call it a
**true blocker**. Ask the user only when code/repo evidence cannot answer it. Everything else is
the orchestrator's responsibility.

### 3. Protect user direction

Challenge scope and surface tradeoffs, but do not silently replace a user's stated product
direction with your preference. Separate:

- **Technical decision:** choose the safest evidence-backed implementation.
- **Product/taste decision:** recommend options, explain impact, and preserve the user's stated
  direction unless they approve a change.
- **Feasibility/security blocker:** state it clearly and redesign around it when possible.

### 4. Prefer bounded mergeable units

Normal implementation tasks should fit inside `--max-task-days` and preferably 1–3 engineer
days. A task should have one coherent outcome, one primary owner, a reviewable diff, and its own
behavior-level tests. Split larger tasks along contract seams, not arbitrary file counts.

Do not split so aggressively that every task becomes plumbing. If two pieces cannot be tested or
reasoned about independently without sharing internals, they may belong in the same task.

### 5. Tests travel with behavior

Do not create a generic "add tests later" task for behavior that can be tested by the task that
introduces it. Unit, component, integration, contract, and regression obligations belong to the
behavior-owning packet. Dedicated E2E/integration-checkpoint tasks are reserved for flows that
span independently owned components or deployment boundaries.

### 6. Dependencies are contracts, not arrows

`TASK-08 depends on TASK-07` is incomplete. Every dependency edge must point to a named contract
that explains exactly what is consumed and how compatibility is verified.

A dependency may constrain **start**, **merge**, or **runtime** independently. Use this to safely
parallelize work:

- A consumer may **start** once its consumed contract is frozen and a fixture/mock exists.
- A consumer may need the provider implementation before it can **merge** or pass integration.
- A task may have a **runtime** dependency even when implementation work is parallel.

Do not serialize work merely because runtime components call each other.

---

# Process — execute in order

## Phase 0 — Establish project intent

Write a concise project charter before decomposition:

- Project goal and user-visible outcome.
- Who/what is affected.
- Current behavior, verified from code/docs when possible.
- Desired behavior.
- Why the work exists now.
- Success measures.
- In-scope and explicitly out-of-scope behavior.
- Hard constraints: compatibility, platform, security, data, deadlines, dependencies.
- Existing decisions that must be preserved.

If the request is already precise, do not interrogate the user for information the repository can
provide. Search first.

## Phase 1 — Repository reconnaissance

Build a codebase evidence map. At minimum inspect the areas that determine:

- entry points and runtime boundaries,
- relevant domain models and storage,
- public/internal APIs,
- auth and permissions,
- state machines or async processing,
- shared libraries/utilities,
- test layout and test commands,
- CI/build/lint/typecheck commands,
- deployment/runtime configuration,
- observability and error handling,
- migrations/backfills if data changes,
- neighboring implementations worth reusing.

Record exact paths and important symbols. Distinguish **verified fact**, **inference**, and
**proposed change**. Do not use a proposed path as evidence that it already exists.

### Reconnaissance output

Produce a table:

| Area | Existing source of truth | Relevant symbols/contracts | Why it matters |
|---|---|---|---|

Also list repository commands you actually verified, such as the real unit-test, integration,
lint, typecheck, build, migration, and E2E commands. Never write placeholder commands like
`npm test` if the repository uses something else.

## Phase 2 — Product and scope review

Apply the `/office-hours` and `/plan-ceo-review` lenses:

- Is the project solving the stated problem?
- Is any scope accidental, duplicate, or already implemented?
- What is the smallest complete product slice?
- What neighboring behavior is in the blast radius and must be handled to avoid a partial fix?
- Which requests are separate projects and should not contaminate this DAG?
- What user-visible acceptance outcomes matter more than implementation details?

Record accepted scope changes. Preserve unresolved product/taste choices as explicit decisions,
not assumptions hidden inside tasks.

## Phase 3 — Lock architecture

Apply the `/plan-eng-review` lens before creating implementation tasks.

Create an **Architecture Decision Register**. Every material decision gets an ID (`ADR-01`,
`ADR-02`, ...), status, choice, evidence/rationale, alternatives rejected, consequences, and the
contracts/tasks affected.

Lock, where relevant:

- component/service boundaries,
- data ownership and schema shape,
- API/command/event boundaries,
- sync vs async behavior,
- state transitions,
- idempotency/retry rules,
- consistency/transaction boundaries,
- caching and invalidation,
- authentication/authorization,
- privacy and secrets handling,
- migration/backfill strategy,
- failure and partial-failure semantics,
- observability/SLO expectations,
- backwards compatibility/versioning,
- rollout/feature-flag strategy,
- rollback/recovery.

The architecture is "locked" when task authors no longer need to choose among competing designs
while implementing ordinary packets. If implementation discovers evidence that invalidates an
ADR, it must be escalated as a contract change, not quietly improvised.

## Phase 4 — Establish system invariants

Create numbered invariants (`INV-01`, `INV-02`, ...). These are properties that must remain true
across task boundaries. Examples include authorization rules, uniqueness guarantees, ordering,
idempotency, privacy constraints, lifecycle rules, and compatibility requirements.

For every invariant specify:

- statement,
- owner/component,
- enforcing mechanism,
- tasks that may affect it,
- verification/test that proves it.

If an invariant has no enforcement or verification path, it is only an aspiration. Fix it.

## Phase 5 — Build the component graph

Map components before tasks. Each component needs:

- responsibility,
- source-of-truth paths,
- data it owns,
- interfaces it provides,
- interfaces it consumes,
- invariants it enforces,
- likely workstream owner.

Render a Mermaid component graph when more than one component is involved. Label meaningful
edges with the interface or data exchanged.

Do not use implementation tasks as components. `API`, `auth domain`, `web client`, `worker`,
`database`, and `event consumer` can be components. `Add endpoint` is a task.

## Phase 6 — Create the contract registry

Contracts are first-class project artifacts. Assign stable IDs such as `CONTRACT-AUTH-01`.

Every contract record MUST contain:

1. **Contract ID and name**
2. **Provider component and provider task**
3. **Consumer component(s) and consumer task(s)**
4. **Contract type** — API, function/interface, schema, event, queue message, file format, CLI,
   config, storage, migration, UI state, or other explicit boundary
5. **Source of truth** — exact existing path/symbol or **CREATE** path
6. **Inputs** — fields/types/constraints/defaults
7. **Outputs** — fields/types/constraints/nullability
8. **Preconditions**
9. **Postconditions**
10. **Error semantics** — status/error codes, retryability, user-visible behavior
11. **Idempotency/retry semantics**
12. **Ordering/concurrency semantics**, when relevant
13. **Security/authorization semantics**
14. **Lifecycle/versioning/backwards-compatibility rule**
15. **Fixture/mock/stub** available to consumers
16. **Freeze point** — the task/milestone after which consumers may rely on it
17. **Change protocol** — what happens if the provider must change it after freeze
18. **Verification** — contract test or exact check proving provider and consumer agree

### Contract freeze rule

A contract is not frozen merely because someone wrote an interface name in a ticket. Freeze only
when its shape, failure semantics, compatibility rule, and consumer fixture are documented well
enough for independent implementation.

After freeze, breaking changes require:

1. identify all consumers,
2. update the contract record,
3. update fixtures/mocks,
4. update affected task packets and DAG edges,
5. re-run contract validation,
6. re-sequence work if required.

No provider may "just change the response" after downstream work has started.

## Phase 7 — Build the contract-first dependency DAG

Now create tasks and edges. The graph MUST be a DAG. A directed edge is valid only when it names
one or more contracts or an explicit non-interface prerequisite such as a migration/infra gate.

Represent each edge with:

- `from` task,
- `to` task,
- `contract_ids`,
- `start_dependency` — what must be frozen/available before consumer work starts,
- `merge_dependency` — what must exist before the consumer can merge,
- `runtime_dependency` — what must exist in production,
- `verification` — test/check that proves the handoff.

Example:

```yaml
- from: TASK-07
  to: TASK-08
  contract_ids: [CONTRACT-AUTH-01]
  start_dependency: frozen contract + LoginResponse fixture
  merge_dependency: provider contract tests passing
  runtime_dependency: auth API deployed and reachable
  verification: web auth adapter contract suite
```

### DAG validation

Before continuing, prove all of the following:

- no cycles,
- every edge has a contract or explicit gate,
- every referenced contract has exactly one source-of-truth owner,
- every consumer appears in the corresponding contract record,
- no task starts before required contracts are frozen,
- no hidden shared mutable artifact creates an undeclared dependency,
- no two same-wave tasks have overlapping write ownership unless intentionally serialized,
- critical path is identified,
- independent work is parallelized where contract fixtures make that safe.

If the graph is cyclic, do not hide the cycle with numbering. Fix the architecture or extract a
shared foundation/contract task.

## Phase 8 — Form workstreams and execution waves

Group tasks into workstreams around ownership boundaries, not job titles. Examples: data
foundation, identity, API, web client, worker/events, observability, deployment.

Create execution waves (`W0`, `W1`, ...):

- Tasks in the same wave should be safe to execute in parallel.
- A task may begin in the same wave as its provider implementation when the consumed contract was
  frozen earlier and a fixture/mock exists.
- Shared-file edits need an explicit owner or a serial merge order.
- Each convergence point across workstreams gets an integration checkpoint.

Show both:

1. **start order** — when implementation may begin,
2. **merge/integration order** — when work may land and be exercised together.

This distinction is mandatory for meaningful parallelization.

## Phase 9 — Write implementation-ready task packets

Use `/spec`-level precision for every task. A normal task packet MUST contain all fields below.
If a field is not applicable, write `N/A — <reason>` rather than omitting it.

### Required task packet schema

```markdown
## TASK-XX — <Outcome-oriented title>

**Objective**
One observable outcome.

**Why this task exists**
What project capability or risk this unit owns.

**Workstream / execution wave**
<workstream>, W<N>

**Depends on**
- TASK-YY via CONTRACT-...

**Blocks / consumers**
- TASK-ZZ via CONTRACT-...

**Inputs**
- Exact contracts, schemas, fixtures, decisions, or existing code relied on.

**Outputs**
- Concrete code/data/config/contracts created or changed.

**Owned files**
- `existing/path` — exact intended responsibility
- **CREATE** `new/path` — reason

**Non-owned / do-not-touch files**
- `path` — owned by TASK-...

**Contracts consumed**
- CONTRACT-... — exact assumptions used

**Contracts produced**
- CONTRACT-... — exact handoff exposed

**Implementation steps**
1. Ordered, technically specific step.
2. ...

**Invariants affected**
- INV-... — how this task preserves/enforces it

**Failure/error semantics**
- Expected failures and behavior.

**Security/privacy**
- Auth, authorization, secret, PII, input-validation implications.

**Observability**
- Logs, metrics, traces, alerts, audit events, or N/A with reason.

**Migration/backfill/compatibility**
- Forward + backward compatibility and rollout needs.

**Required tests**
- Unit: exact behavior cases
- Integration: exact boundary cases
- Contract: provider/consumer conformance cases
- E2E: only if this task owns a complete flow
- Regression: existing behaviors that must remain green

**Verification commands**
- Commands verified from the repository. No invented generic command.

**Acceptance criteria**
1. PASS/FAIL observable criterion.
2. PASS/FAIL observable criterion.

**Rollback / recovery**
- How to revert or safely disable this unit.

**Downstream handoff**
- Frozen contract(s), fixtures/mocks, error codes, examples, and anything the consumer is allowed
  to assume. Explicitly state what the consumer must NOT need to know about internals.

**Definition of done**
- Code + tests + contract artifacts + docs/config required by this task are complete.
- Acceptance and verification gates pass.
- Every produced contract is frozen or explicitly marked not-yet-consumable.
- Downstream tasks can proceed from the documented handoff without reading this implementation.

**Effort**
- <0.5d / 1d / 2d / 3d> with main cost driver.
```

### File ownership rules

- Prefer exact files over broad directories when the planned diff can be known.
- One task owns a shared file for a wave. Other tasks consume its resulting contract.
- If multiple tasks truly must edit the same file, serialize them or extract a shared task.
- A task may read non-owned files. It may not casually edit them.
- Generated files belong to the task that changes their source-of-truth generator unless the repo
  explicitly treats regeneration as a separate atomic operation.

### Acceptance criteria rules

Acceptance criteria must be externally observable or mechanically verifiable. Avoid phrases like
"works correctly," "handles edge cases," "is robust," or "tests added" without enumerating what
passes and what behavior is expected.

Number every criterion so reviewers can refer to failures precisely.

## Phase 10 — Apply the downstream-consumer test

For **every provider → consumer edge**, answer this exact question:

> Can the consumer implement and test its task using only the documented contract, fixture/mock,
> inputs/outputs, invariants, and error semantics, without reading provider internals?

- **YES:** record the evidence: contract ID + fixture + verification test.
- **NO:** the plan is not ready. Add missing contract detail, move responsibility, extract a
  contract/foundation task, or combine tasks that are falsely separated.

Do not waive this test because both tasks are assigned to the same person or agent.

## Phase 11 — Define integration checkpoints

Whenever two or more independent workstreams converge, add an explicit integration checkpoint
(`INT-01`, `INT-02`, ...). It is a real DAG node/gate, not a sentence saying "integrate later."

Each checkpoint states:

- upstream tasks/contracts required,
- environment/fixture needed,
- cross-component scenarios to exercise,
- negative/failure scenarios,
- compatibility/migration checks,
- exact commands,
- artifacts/logs expected,
- owner of any integration defect,
- exit criteria for unlocking downstream waves.

Integration checkpoints should diagnose contract mismatches early. Do not postpone all
cross-component verification to final E2E.

## Phase 12 — Build the project test matrix

Create one matrix mapping requirements and contracts to test layers.

| Requirement / contract | Unit | Integration | Contract | E2E | Regression | Owning task/checkpoint |
|---|---|---|---|---|---|---|

Rules:

- Every invariant has at least one verification path.
- Every contract edge has a contract/conformance check.
- Every critical user journey has an E2E path at the appropriate convergence point or release
  gate.
- Every bug/regression risk named during planning has an explicit regression case.
- Security-sensitive boundaries include authorization/negative tests.
- Retry/idempotency semantics include duplicate/replay tests where relevant.
- Migrations include forward, mixed-version/compatibility, and rollback/recovery verification when
  the deployment model requires them.

## Phase 13 — Release gate

Create a final `RELEASE-GATE` node that consumes the completed integration checkpoints. It must
cover what is relevant to the project:

- required unit/integration/contract/E2E/regression suites,
- lint/typecheck/build,
- migration readiness and rollback,
- feature flag/config readiness,
- security/privacy checks,
- observability dashboards/alerts/log verification,
- performance/capacity checks when behavior could regress them,
- deployment sequencing,
- canary/smoke verification if supported,
- rollback trigger and owner,
- documentation/runbook/support changes required for operation.

Do not invent infrastructure that the repository does not use. Mark irrelevant gates `N/A` with
repo evidence.

---

# Required final output

The final answer/plan MUST use this structure in this order.

## 1. Project charter
Goal, users, current/desired behavior, success measures, scope, constraints.

## 2. Codebase reconnaissance
Exact paths/symbols/commands and verified facts that ground the plan.

## 3. Architecture Decision Register
ADR table with locked technical decisions and any true unresolved blockers.

## 4. System invariants
Numbered invariant registry with enforcement and verification.

## 5. Component graph
Mermaid graph plus a concise component ownership table.

## 6. Contract registry
Full contract records. Do not hide key fields in prose elsewhere.

## 7. Dependency DAG
Mermaid task DAG. Label edges with contract IDs. Also provide a machine-readable edge table with
start/merge/runtime semantics.

## 8. Workstreams and execution waves
Parallel lanes, start order, merge order, and sequencing rationale.

## 9. Task index
Compact table: ID, outcome, workstream, wave, dependencies, contracts, effort, critical-path flag.

## 10. Full implementation packets
One complete packet per task using the required schema.

## 11. Integration checkpoints
All convergence gates with scenarios and exit criteria.

## 12. Test matrix
Requirement/invariant/contract coverage across test layers.

## 13. Critical path and parallelization plan
State what controls elapsed time, what can run concurrently, and why that concurrency is safe.

## 14. Release, migration, rollback, and observability gates
Operational completion requirements.

## 15. Risks and true blockers
Only unresolved items that materially affect execution. Do not convert ordinary engineering work
into "risks" to avoid deciding it.

## 16. Delegation readiness report
A final PASS/FAIL checklist proving every task and edge is consumable by an engineer who did not
design the system.

---

# Delegation readiness gate — HARD GATE

Do not call the orchestration plan ready until every item passes.

### Task completeness

- [ ] Every task has an objective and bounded outcome.
- [ ] Every task has explicit inputs and outputs.
- [ ] Every task has dependencies and consumers, or explicitly says none.
- [ ] Every task has owned files and non-owned/do-not-touch boundaries.
- [ ] Every existing file path was verified; every new path is marked **CREATE**.
- [ ] Every task lists consumed and produced contracts.
- [ ] Every task maps affected invariants.
- [ ] Every task defines failure/error semantics.
- [ ] Every task includes security/privacy and observability impact or justified N/A.
- [ ] Every task includes migration/compatibility impact or justified N/A.
- [ ] Every behavior-owning task has its test obligations, not a deferred generic test ticket.
- [ ] Every task has exact, repository-verified verification commands.
- [ ] Every task has numbered PASS/FAIL acceptance criteria.
- [ ] Every task has rollback/recovery guidance.
- [ ] Every task has a downstream handoff.
- [ ] Every normal task fits the effort cap or documents an atomic exception.

### Contract completeness

- [ ] Every dependency edge names a contract or explicit gate.
- [ ] Every contract has one source-of-truth owner.
- [ ] Every contract names all known consumers.
- [ ] Inputs, outputs, constraints, pre/postconditions, and errors are explicit.
- [ ] Retry/idempotency/order/security semantics are explicit where relevant.
- [ ] Compatibility/versioning behavior is explicit.
- [ ] A fixture/mock/stub exists or is created before consumers start.
- [ ] Freeze point and post-freeze change protocol are explicit.
- [ ] Provider/consumer conformance verification exists.
- [ ] Every provider → consumer pair passes the downstream-consumer test.

### DAG and parallelism

- [ ] DAG is acyclic.
- [ ] Start, merge, and runtime dependencies are distinguished where useful.
- [ ] Same-wave write ownership does not conflict.
- [ ] Shared artifacts have explicit owners.
- [ ] Contract freezing enables parallel consumer work where safe.
- [ ] Every convergence across workstreams has an integration checkpoint.
- [ ] Critical path is identified.
- [ ] Parallel lanes do not depend on hidden implementation knowledge.

### Project quality

- [ ] Architecture decisions are locked or true blockers are explicit.
- [ ] Every invariant has enforcement + verification.
- [ ] Test matrix covers contracts, critical journeys, regressions, and negative cases.
- [ ] Release gate includes repository-relevant operational checks.
- [ ] Scope includes the complete blast radius but excludes unrelated projects.
- [ ] The plan leaves no ordinary design choice to a downstream implementer.

If any checkbox fails, report **NOT READY** and repair the plan before issue creation.

---

# Issue creation mode (`--create-issues`)

Only enter this mode after the delegation readiness gate passes.

Create:

1. one **epic/parent issue** containing the charter, architecture summary, invariants, component
   graph, DAG, contract index, execution waves, critical path, integration checkpoints, release
   gate, and links to all children;
2. one **child issue per implementation packet**;
3. explicit **integration-checkpoint issues** when they require execution rather than being a CI
   gate inside a normal task;
4. a **release-gate issue** when the project requires coordinated release work.

Use native parent/child or dependency features if the tracker supports them. Otherwise preserve
hierarchy and DAG links in issue bodies with stable task IDs and checklists. Do not pretend a flat
issue list is a dependency graph.

### Child issue body requirements

Each child issue contains the full task packet. It must not rely on the epic for details required
to implement the task. Link the epic for context, but duplicate the task's consumed/produced
contract details needed for independent execution.

### Creation order

1. Create the epic.
2. Create foundation/contract tasks first so issue numbers can be linked by consumers.
3. Create remaining tasks in topological order.
4. Backfill cross-links/dependency references if tracker IDs were unknown at draft time.
5. Re-read created issues and verify no packet or contract field was lost.
6. Publish a final mapping of `TASK-ID → tracker issue` and `CONTRACT-ID → provider/consumers`.

Issue creation is not completion. The orchestration is complete only when the created tracker
representation still passes the delegation readiness gate.

---

# Anti-patterns — reject these

- **Flat backlog:** a numbered list with no dependency graph.
- **Arrow-only DAG:** edges say "depends on" but do not define consumed contracts.
- **Mega-ticket:** "build backend" or "implement feature" spanning several natural contracts.
- **File-sliced work:** separate tasks for model/controller/test when they cannot deliver behavior
  independently.
- **Tests-later ticket:** behavior ships in one task and its basic tests are postponed to another.
- **Hidden shared-file collision:** parallel tasks both plan to edit the same central file.
- **Mockless parallelism:** consumer starts before provider, but no frozen contract or fixture
  exists.
- **Implementation leakage:** consumer acceptance requires reading provider private classes,
  database internals, or undocumented side effects.
- **Architecture by junior:** task says "choose a queue/database/library/pattern" without a prior
  ADR when the choice is material.
- **Invented precision:** fake file paths, test counts, commands, schemas, or effort estimates not
  grounded in the repository.
- **Everything-is-a-blocker:** ordinary engineering decisions pushed back to the user.
- **E2E-at-the-end only:** no integration checkpoints until release.
- **Contract drift:** provider changes an interface after freeze without updating consumers,
  fixtures, tests, and DAG sequencing.

---

# Quality bar

A successful `/project-orchestrator` output should allow a director to hand different tasks to
multiple junior engineers or agents and expect them to converge without private coordination.
The plan itself is the coordination mechanism: architecture decisions, invariants, contracts,
ownership, fixtures, tests, integration checkpoints, and release gates carry the knowledge.

The strongest signal of quality is not the number of tickets. It is this:

> **Any ready downstream task can be implemented from its packet and consumed contracts alone,
> and independently completed tasks rejoin at explicit integration gates without surprise.**
