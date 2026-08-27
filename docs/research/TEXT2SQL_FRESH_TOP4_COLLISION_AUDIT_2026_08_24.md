# Fresh Text2SQL Top-Candidate Collision Audit (2026-08-24)

> Status: literature-first research spike. No implementation changes.
> Purpose: stress-test the strongest directions from `TEXT2SQL_FRESH_FRONTIER_AND_BATCH_IDEAS_2026_08_24.md` against the closest 2025–2026 prior work, then rewrite or kill ideas that collide.

## 1. Audit rule

A candidate survives only if its core claim cannot be reduced to a known mechanism with Text2SQL terminology substituted in. In particular, this pass treats general agent routing, SQL probing, semantic-operator optimization, and branchable agent environments as prior art rather than novelty.

For each candidate we record:

- nearest collision;
- what is already occupied;
- exact surviving gap;
- smallest publishable thesis;
- killer experiment;
- falsification condition;
- likely reviewer objection;
- revised priority.

---

# 2. Candidate A — MetaSQL: adaptive scaffold routing

## 2.1 Collision audit

The broad claim “different tasks need different reasoning paradigms, so route between them” is **not new**.

### General-agent routing is already strong

**Select-then-Solve (2026)** compares Direct, CoT, ReAct, Plan-Execute, Reflection, and ReCode over four frontier LLMs and ten benchmarks. No paradigm dominates; oracle per-task routing beats the best fixed paradigm by 17.1 percentage points on average. A lightweight learned router improves over the best fixed paradigm.

Reference: Zhou et al., *Select-then-Solve: Paradigm Routing as Inference-Time Optimization for LLM Agents*, 2026. https://arxiv.org/abs/2604.06753

**RouteMoA (ACL 2026)** dynamically selects model/agent subsets before inference and explicitly optimizes performance, cost, and latency.

Reference: Wang et al., *RouteMoA*, ACL 2026. https://aclanthology.org/2026.acl-long.558/

**MoMA (2025)** already unifies model and agent routing, including context-aware agent selection.

Reference: Guo et al., *Towards Generalized Routing: Model and Agent Orchestration for Adaptive and Efficient Inference*, 2025. https://arxiv.org/abs/2509.07571

Session-aware routing work in 2026 also shows that long-horizon agent routing must account for tool-loop continuity and switching costs, not only the current prompt.

Therefore the naive claim

> “route Text2SQL questions to different agent architectures”

is insufficient by itself.

## 2.2 Why Text2SQL still offers a distinct gap

The fresh Text2SQL literature exposes an unusually clean natural experiment:

- Spider 2.0 rewards substantial enterprise exploration;
- BIRD-Interact requires long interactive/tool trajectories;
- SQL debugging is a different action space;
- AI-native SQL (Spider 2.0-AIFunc) reports that heavy traditional Text2SQL scaffolding can fail to transfer and that minimal agent setups can be competitive or better;
- simple classical tasks can often be solved directly.

The important unit of routing is therefore not merely **model** or generic **reasoning paradigm**. It is a *database reasoning harness* with materially different access to evidence and verification:

```text
H0 direct SQL
H1 schema-retrieval generator
H2 self-driven real-DB explorer
H3 multi-candidate + distinguishing-world verifier
H4 memory-augmented repair
H5 interactive ask/inspect/execute agent
H6 AI-native semantic-operator planner
H7 SQL debugging/remediation agent
```

Each harness has different failure opportunities, database/tool costs, and evidence channels.

## 2.3 Surviving thesis: Database-Harness Routing

> **A Text2SQL system should select an evidence/verification harness conditionally from the task and early trajectory, rather than fixing one agent scaffold for every SQL regime.**

The scientific contribution must be the **routing state and evaluation problem**, not a new router architecture.

Candidate state features:

- schema scale / join-graph complexity;
- ambiguity over schema links;
- predicted relational vs AI-operator complexity;
- availability/reliability of documentation;
- first-pass parse/dry-run result;
- execution anomaly signatures;
- user-interaction permission;
- query risk (read / write / remediation);
- estimated marginal value of more evidence.

## 2.4 Stronger benchmark contribution: No-One-SQL-Agent

Construct a unified evaluation matrix where the *same harness portfolio* is tested on:

1. conventional Text2SQL;
2. large-schema enterprise Text2SQL;
3. interactive/CRUD tasks;
4. SQL debugging;
5. query-efficiency tasks;
6. AI-native SQL functions.

For each task `x`, let `h*(x)` be the best harness under a common budget. Define:

```text
routing regret(x) = utility(h*(x), x) - utility(router(x), x)
```

with utility combining correctness, LLM cost, database/tool cost, latency, and human turns.

### Killer experiment

First measure the **oracle harness gap**. If an oracle per-task harness selector barely improves over the best fixed harness, there is no routing paper.

Only if the oracle gap is substantial should a learned router be built.

### Falsification condition

Kill or downgrade the idea if:

- the oracle harness improves <2 absolute points under cost-matched evaluation; or
- one simple explorer dominates across all fresh regimes; or
- routing gains disappear after equalizing compute/tool access.

### Reviewer objection

“This is Select-then-Solve applied to SQL.”

### Required answer

The paper must demonstrate that routing decisions depend on **database evidence access and verification semantics**, not generic CoT/ReAct choice, and that the cross-regime Text2SQL benchmark reveals architecture reversals absent from generic routing benchmarks.

## Revised score

- Novelty: 4/5 (not 5/5 after general routing collision)
- Feasibility: 5/5
- Scientific clarity: 5/5
- Priority: **Tier A**

---

# 3. Candidate B — Discriminative ProbeSQL

## 3.1 Collision audit: “SQL probes” are already occupied

The phrase “let the model generate SQL probes to understand the database” is no longer novel.

**SDE-SQL (ACL 2026)** explicitly introduces self-driven database exploration via generated and executed SQL probes and reports an 8.02% relative execution-accuracy gain over its vanilla open-model baseline.

Reference: Xie et al., *SDE-SQL: Enhancing Text-to-SQL Generation in Large Language Models via Self-Driven Exploration with SQL Probes*, ACL 2026. https://aclanthology.org/2026.acl-long.116/

**APEX-SQL** similarly uses hypothesis-verification loops and empirical database exploration.

Thus the naive formulation is rejected.

## 3.2 Synthetic distinguishing worlds are also occupied

A second possible formulation — “construct data to distinguish candidate SQLs” — is heavily occupied:

- **DPC (ACL 2026)** creates a Minimal Distinguishing Database and uses an independent Python/Pandas solver for candidate selection.
- **SpotIt (ICLR 2026)** uses bounded formal verification to search for a database that differentiates predicted and gold SQL.
- **ParSEval (PVLDB 2025)** generates plan-aware test databases that exercise logical-plan branches and expose inequivalence.

References:
- DPC: https://aclanthology.org/2026.acl-long.313/
- SpotIt: https://proceedings.iclr.cc/paper_files/paper/2026/hash/70e692da44c19710386648694e2b899b-Abstract-Conference.html
- ParSEval: DOI 10.14778/3749646.3749727

Therefore this candidate survives only if it studies a different information-acquisition problem.

## 3.3 Surviving thesis: Real-Database Bayesian Experimental Design

The remaining gap is to treat the **real database as an experimental system** and choose the next *legal observational query* to maximally distinguish uncertain semantic hypotheses.

Example:

```text
User: revenue by customer last quarter

H1: revenue = invoice.total
H2: revenue = captured payment - refund
H3: revenue = recognized line-item revenue

Belief: P(H1)=.35, P(H2)=.40, P(H3)=.25
```

The agent may choose among probes such as:

```text
P1: overlap of invoice and payment populations
P2: distribution of refunds by payment state
P3: whether line items reconcile to invoice totals
P4: value/domain sample for revenue-status fields
P5: schema/document observation
```

A probe should be selected by expected posterior discrimination or expected decision utility, not by “what looks useful” in free-form reasoning.

Formally:

```text
probe* = argmax_p E[U(posterior after result(p))] - lambda * cost(p)
```

This differs from:

- SDE-SQL: probes are self-generated exploration actions but not framed as optimal experimental design over explicit hypotheses;
- DPC/SpotIt/ParSEval: they synthesize alternative databases to distinguish programs, whereas this proposal chooses observations on the *actual database* to resolve semantic uncertainty before final SQL generation;
- user-clarification EIG: the observation source here is machine-accessible database/schema/document evidence, not only a human answer.

## 3.4 Minimal paper: ActiveProbeSQL

Start with a controlled environment where every task provides 2–4 plausible semantic hypotheses and a library of legal probes with known result distributions.

Compare:

1. no probes;
2. fixed probe sequence;
3. SDE-SQL-style free exploration;
4. greedy heuristic exploration;
5. entropy/information-gain selection;
6. value-of-information selection including query cost.

Primary metrics:

- final execution/semantic accuracy;
- probes per solved task;
- DB rows scanned / estimated warehouse cost;
- hypothesis entropy reduction per probe;
- success per unit of observation cost.

### Killer experiment

Construct matched tasks where many plausible probes are available but only one or two distinguish the actual semantic ambiguity. If explicit experimental design cannot beat free exploration under equal probe cost, stop.

### Falsification condition

Downgrade if information-gain/EVPI selection yields <10% reduction in probe count at matched accuracy, or if the LLM cannot maintain a calibrated enough hypothesis set for selection to matter.

### Reviewer objection

“This is SDE-SQL with an entropy score.”

### Required answer

The paper needs controlled ambiguity annotations, explicit posterior updates, probe-cost accounting, and a demonstration that optimized *choice of experiment* matters independently of simply allowing more exploration.

## Revised score

- Novelty: 4/5
- Feasibility: 4/5
- Scientific clarity: 5/5
- Priority: **Tier A**

---

# 4. Candidate C — StochasticSQL / AI-native SQL

## 4.1 Collision audit: optimization and quality guarantees are already crowded

This candidate was initially over-ranked. The database-systems literature has moved very quickly.

**LOTUS / Semantic Operators** already treats model-backed filters, joins, maps, and aggregations as declarative operators with alternative physical implementations and statistical accuracy guarantees.

Reference: Patel et al., *Semantic Operators: A Declarative Model for Rich, AI-based Analytics Over Text Data*, 2024/2025. https://arxiv.org/abs/2407.11418

**Stretto (2026)** explicitly optimizes end-to-end runtime–accuracy trade-offs, jointly selects operator implementations, allocates error budgets, and provides end-to-end quality guarantees.

Reference: Sanmartino et al., *The Stretto Execution Engine for LLM-Augmented Data Systems*, 2026. https://arxiv.org/abs/2602.04430

**Sema (2026)** treats LLM semantic operators as first-class query-engine citizens and uses adaptive query execution to reorder/fuse operations and balance token cost/latency under accuracy constraints.

Reference: Qi et al., *Sema*, 2026. https://arxiv.org/abs/2603.11622

**iPDB (2026)** supports semantic project/select/join/group operations inside relational execution with semantic query optimizations.

Reference: Kumarasinghe et al., *iPDB*, 2026. https://arxiv.org/abs/2601.16432

**SemJoin (2026)** dynamically routes semantic joins among execution strategies based on table/predicate characteristics.

Reference: Gou et al., *SemJoin*, 2026. https://arxiv.org/abs/2606.29532

Production systems from Google and Snowflake also already optimize AI predicates aggressively because LLM inference dominates cost.

Therefore the following are **not fresh enough** as standalone papers:

- reorder AI predicates;
- choose smaller/larger model per semantic operator;
- optimize semantic join strategy;
- allocate error budget across semantic operators;
- proxy prefiltering for cost alone.

## 4.2 A narrower surviving gap: longitudinal semantic reproducibility

Current systems mainly optimize a query *given* an operator/model behavior distribution. Text2AIFunc introduces a second problem:

> The natural-language request is compiled into SQL that invokes externally evolving model-backed operators whose semantics can change after the query text itself is frozen.

A traditional SQL query is expected to remain semantically stable under ordinary engine upgrades. An AI-native query may change because of:

- endpoint model upgrade;
- system-prompt/rubric revision;
- embedding/model checkpoint change;
- provider-side decoding changes;
- prompt-template normalization;
- retry/parallelism policy;
- non-deterministic inference.

The research object is not only per-run statistical accuracy; it is **semantic reproducibility across time and operator versions**.

## 4.3 Candidate paper: DriftAIFunc / Semantic Query Contracts

Attach a reproducibility contract to every generated AI-SQL query:

```text
operator family
model/version fingerprint
natural-language predicate hash
quality metric + tolerance
reference calibration set
allowed answer drift
retry/aggregation policy
validity interval
```

When the underlying AI operator changes, run a small calibration suite and decide whether the stored query remains semantically equivalent enough for production use.

### Killer experiment

Take a fixed set of Spider2-AIFunc-like queries and execute them across:

- repeated runs;
- multiple function/model versions;
- controlled prompt/rubric rewrites;
- model sizes;
- time-separated snapshots.

Measure how often a query judged “correct” once violates its semantic contract later.

Compare:

- query text only;
- pinned model where possible;
- simple repeated voting;
- calibration-set contract;
- adaptive revalidation.

### Primary endpoint

`semantic contract violation rate` at a fixed revalidation cost.

### Falsification condition

If repeated/version-shift execution produces negligible decision-level drift on realistic AI-SQL tasks, the problem is too weak.

### Reviewer objection

“LOTUS/Stretto already provide statistical guarantees.”

### Required answer

Distinguish *within-version execution-quality guarantees* from **longitudinal query semantics under externally evolving model operators**.

## Revised score

- Novelty: 4/5 only under the reproducibility/drift formulation
- Feasibility: 3/5 because platform/version access is difficult
- Scientific clarity: 4/5
- Priority: **Tier B**

---

# 5. Candidate D — Branch-and-Verify CRUD

## 5.1 Collision audit: branching itself is occupied

The broad statement “agents should branch database state before risky writes” is already explicitly part of the 2026 Agentic Data Environments agenda.

**BranchBench** models repeated branch–mutate–evaluate loops and shows current branchable DBMSes are not ready for agentic workloads: systems optimized for branching can suffer large read slowdowns while query-optimized systems pay very high branch creation costs.

Reference: Ang et al., *BranchBench: Aligning Database Branching with Agentic Demands*, 2026. https://arxiv.org/abs/2604.17180

**Agentic Data Environments** explicitly frames branchable database/system state as a safety primitive for speculative autonomous actions.

Reference: Ang et al., *Agentic Data Environments*, 2026. https://arxiv.org/abs/2607.07397

**StateFork/Waypoint** extends this idea beyond DBMS state to whole agent environments.

**A Simple and Fast Way to Handle Semantic Errors in Transactions** (2024) also studies removable LLM-generated transactions while preserving database consistency via invariant-based coordination.

Reference: Zeng et al., https://arxiv.org/abs/2412.12493

Additionally, **DBA-Bench (2026)** evaluates database-operation agents in persistent read/write PostgreSQL environments with safety-constrained outcomes; the best automated Safe Pass is reported far below the human DBA reference.

Reference: Chen et al., *DBA-Bench*, 2026. https://arxiv.org/abs/2607.22165

Thus “branch first, then evaluate” is infrastructure prior art.

## 5.2 Surviving gap: semantic effect verification

What remains underdeveloped in Text2SQL/CRUD is a formal bridge from the **natural-language requested action** to an **expected state transition**.

Example request:

> Move customer C123 from trial to active, preserving billing history and creating one activation audit record.

Compile it before SQL generation into:

```yaml
must_change:
  customer.status:
    key: C123
    from: trial
    to: active

must_create:
  activation_audit:
    count: 1
    customer_id: C123

must_preserve:
  billing_history: all existing rows
  invoice.total_paid: unchanged

must_not_change:
  other_customers: true

cardinality:
  max_existing_rows_updated: 1
```

Generate one or more SQL workflows, execute them on a branch/transaction, compute actual relational delta, and compare it with the contract.

Here branching is only an execution primitive. The paper contribution is **intent-to-effect compilation + effect verification**.

## 5.3 Candidate paper: DeltaSQL / Semantic Delta Contracts

> **For state-mutating database agents, correctness should be defined over the relational state transition requested by the user, not merely whether the generated SQL executes or whether the final natural-language answer sounds correct.**

### Dataset construction

Create controlled CRUD tasks with:

- pre-state snapshot;
- natural-language request;
- executable candidate action(s);
- machine-checkable expected delta;
- forbidden side effects;
- acceptable alternative implementations.

Task families:

- single-row update;
- conditional bulk update;
- insert + audit side effect;
- status transition with invariant;
- multi-table mutation;
- idempotent retry;
- partial failure / compensation;
- concurrent-state conflict.

### Killer experiment

Take agents that pass normal execution/test checks and measure how often they introduce *extra state changes* outside the requested effect.

Compare:

1. SQL execution success only;
2. AST allowlist/safety policy;
3. row-count guard;
4. post-hoc LLM review;
5. semantic delta contract;
6. branch + semantic delta contract.

Primary metrics:

- requested-effect recall;
- forbidden-side-effect rate;
- over-mutation rate;
- under-mutation rate;
- safe task success;
- human-review burden.

### Falsification condition

If ordinary deterministic postconditions or existing benchmark tests already catch nearly every semantically harmful extra mutation, a new contract layer adds little.

### Reviewer objection

“This is ADE/BranchBench plus BIRD-Interact CRUD.”

### Required answer

BranchBench evaluates *branch infrastructure*. BIRD-Interact evaluates task completion. DeltaSQL must evaluate and enforce **natural-language intent as a state-transition specification**, including unwanted but executable side effects.

## Revised score

- Novelty: 5/5 for the semantic-delta formulation
- Feasibility: 4/5 for synthetic/local DB; 3/5 for BIRD-Interact-scale evaluation
- Scientific clarity: 5/5
- Priority: **Tier A+**

---

# 6. Updated ranking after the collision audit

| Rank | Candidate | Updated judgment |
|---|---|---|
| 1 | **DeltaSQL — Semantic Delta Contracts for CRUD agents** | Branching is occupied, but NL intent → machine-checkable state effect remains a clean gap. |
| 2 | **ActiveProbeSQL — real-DB experimental design** | SQL probes are occupied by SDE-SQL; explicit hypothesis-discriminative, cost-aware probe selection still survives. |
| 3 | **Database-Harness Routing / No-One-SQL-Agent** | Generic paradigm routing is occupied; cross-regime evidence/verification-harness routing remains testable and cheap to falsify. |
| 4 | **DriftAIFunc — longitudinal semantic contracts** | AI-query optimization is crowded; reproducibility under model/operator drift is the more defensible slice. |

The most important change is that the original broad versions of **ProbeSQL**, **MetaSQL**, **StochasticSQL**, and **Branch-and-Verify** all partially collide with 2026 work. The surviving contributions are narrower and, consequently, scientifically cleaner.

---

# 7. New ideas generated by the collisions themselves

The collision audit also suggests a second batch of derived directions.

## K1. Oracle-Gap-First Research

Before inventing a router/controller, measure the oracle gap between available actions. If an oracle cannot gain materially from perfect decisions, abort the controller idea before training anything.

Apply this principle to:

- harness routing;
- model routing;
- probe selection;
- memory selection;
- user clarification;
- verification escalation.

This could become a general methodology for avoiding unnecessary agent controllers.

## K2. Observation Compiler

Instead of an unrestricted explorer, compile each uncertainty variable into a typed observation request:

```text
uncertainty: join cardinality
observation: duplicate/fanout probe

uncertainty: value grounding
observation: distinct-value sample

uncertainty: source-of-truth
observation: freshness + overlap + usage probe
```

This offers a middle ground between SDE-SQL free-form probes and fully hand-coded diagnostics.

## K3. Counterfactual Tool-Value Dataset

For a solved trajectory, replay it while removing or replacing one observation/tool call. This creates supervision for the *causal value of a tool call* and can train cost-aware probe/escalation policies.

## K4. Semantic Effect Coverage

Analogous to code coverage: what fraction of requested state-transition obligations has been tested before committing a CRUD action?

A mutation agent should report coverage over:

- target rows;
- target attributes;
- required inserts/deletes;
- preserved invariants;
- forbidden side effects.

## K5. Idempotency-Aware Text2CRUD

Natural-language write requests are often retried by agents. Generate an explicit idempotency contract and evaluate whether repeated execution preserves intended semantics.

## K6. Compensability Score

Before execution, predict whether an action is cleanly reversible. Route high-impact, low-compensability actions to stronger verification/human approval.

## K7. Drift Budget for AI-SQL

Treat semantic drift like an SRE error budget. A saved AI-SQL query accumulates drift evidence over time; once its budget is exhausted, it must be revalidated or recompiled.

## K8. Version-Bisect for Semantic Operators

When an AI-SQL query changes output after a model/platform update, automatically bisect operator versions/prompt components to identify which change caused the semantic regression.

## K9. Harness Failure Attribution

If a heavy agent loses to direct generation, identify whether the loss arose from schema retrieval, probe noise, repair corruption, candidate selection, or context overload. Build controlled harness-component interventions rather than reporting only final accuracy.

## K10. Tool-Call Minimality Certificates

Given a successful SQL trajectory, find a minimal subset of observations/tools sufficient to reproduce the correct answer. Use it as a target for distilling expensive agent behavior into efficient policies.

---

# 8. Recommended research sequence before implementation

The next research-only passes should proceed without building a large system:

1. **DeltaSQL deep audit:** state-transition specifications, transaction invariants, program postconditions, database testing, BIRD-Interact/DBA-Bench task structures. Goal: verify that semantic-effect checking is not already covered by existing CRUD benchmarks.
2. **ActiveProbeSQL deep audit:** Bayesian experimental design, active diagnosis, optimal test selection, SDE-SQL probe mechanics, database cost models. Goal: define a nontrivial controlled benchmark where probe choice — not probe availability — determines success.
3. **Harness-routing oracle study design:** enumerate representative harnesses and task regimes, specify an oracle-gap pilot requiring minimal engineering.
4. **AI-SQL drift audit:** identify platforms/datasets where model/operator versions can be replayed reproducibly; kill the idea if longitudinal experiments cannot be made scientifically controlled.

Only after these passes should an implementation target be selected.
