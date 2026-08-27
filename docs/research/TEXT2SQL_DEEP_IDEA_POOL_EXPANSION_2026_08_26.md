# Deep Text2SQL / Data-Agent Idea Pool Expansion (2026-08-26)

> Status: literature-first research spike; no implementation changes.
> Goal: expand the fresh idea pool with directions that attack deeper structural problems rather than adding another planner / critic / RAG / repair loop.
> Policy: every idea is collision-checked against the 2026 frontier and must expose a falsifiable experiment.

---

# 1. Why another ordinary agent architecture is no longer enough

The 2026 frontier has moved quickly enough that several directions that looked open even a few months ago are already crowded.

- **Database probing is now mainstream.** SDE-SQL gives the model SQL probes; PV-SQL combines probing with rule-based executable verification; PExA builds parallel atomic SQL tests to achieve semantic coverage. Therefore, “let the agent query the database to understand it” is not a standalone contribution.
- **Long-horizon agentic training is occupied.** ReEx-SQL, SQL-Trail, MTSQL-R1 and related RL systems interleave reasoning, execution, verification, memory, and adaptive budgets.
- **Tree-structured SQL correction is occupied.** ACTS-SQL (August 2026) uses plan-guided tree debugging, multiple correction strategies, backtracking, execution verification and clause-level diagnostics.
- **Fine-grained RL rewards are occupied.** EXPO-SQL provides clause-level execution-derived rewards; other work already combines schema, structure, semantic, and execution rewards.
- **Semantic layers are becoming a dominant solution.** A semantic-layer-mediated agent reports 94.15% execution accuracy on Spider2-Snow, while paired benchmarks show that explicit semantic knowledge can add roughly 17–23 percentage points over raw schema prompting.
- **Schema representation itself is now a database-design objective.** Recent work shows that normalization, names, logical views, workload partitions and agent-friendly schema transformations materially affect LLM SQL accuracy.
- **Schema drift and business-rule drift are now explicit benchmarks.** EvoSchema studies schema evolution; LiveSQLBench-Large introduces versioned business-rule drift.
- **Agent/database runtime consistency is becoming a field.** Recent work on semantic isolation and agentic transactions transfers transaction-style guarantees to durable AI workflows.

These collisions suggest a different question:

> **What scientific objects are still missing from the current formulation of Text2SQL?**

This memo focuses on seven missing objects: **time-varying meaning, representation invariance, database provenance, agent observability, evaluation uncertainty, snapshot consistency, and semantic plurality.**

---

# 2. Theme A — Business meaning is a versioned program, not static documentation

## A1. Meaning-Time SQL — bitemporal business semantics

### Problem essence

A database row has a time, but a *business definition* also has a time.

Suppose Q4 2024 revenue was originally defined as:

```text
captured payments - settled refunds
```

and in 2026 finance changes the definition to:

```text
recognized invoice revenue - chargebacks
```

Two legitimate questions now exist:

```text
What was Q4 2024 revenue under the definition finance used in 2024?
```

and

```text
Recompute Q4 2024 revenue under today's finance definition.
```

Current Text2SQL mostly has one temporal axis: data time. LiveSQLBench introduces business-rule drift across releases, but it does not turn **definition time** into a first-class query semantic.

### Mechanism

Borrow bitemporal database semantics.

Each semantic rule carries at least:

```yaml
metric: revenue
valid_from: 2024-01-01
valid_to: 2025-12-31
recorded_at: 2024-01-03
superseded_at: 2026-02-10
formula: ...
```

The semantic plan has two clocks:

```text
data_as_of
meaning_as_of
```

### Falsifiable hypothesis

Explicit bitemporal semantic binding reduces temporal-rule errors on historical analytics compared with simply retrieving the latest or nearest semantic-layer document.

### Killer experiment

For the same historical database snapshot, create several rule versions. Ask minimal pairs that vary only whether the user wants historical meaning or current meaning. Compare:

- full-context LLM;
- latest-rule semantic layer;
- version-retrieval RAG;
- Meaning-Time compiler.

### Novelty judgment

**Very high.** Business-rule drift exists, but *querying over the temporal identity of the business definition itself* is a sharper problem.

---

## A2. Semantic ABI — compatibility rules for business meaning

### Problem essence

Semantic layers are increasingly becoming the interface between agents and physical databases. Once this happens, a metric definition behaves like an API.

But current systems have no rigorous equivalent of:

```text
backward-compatible change
breaking change
major/minor semantic version
```

Changing `revenue` from one formula to another can silently invalidate dashboards, memories, generated SQL, cached answers, and learned agent behavior even if the physical schema is unchanged.

### Mechanism

Define a **semantic application binary interface (ABI)**.

A semantic-layer change is classified by its observable effect on a declared contract domain:

```text
metric formula
allowed dimensions
join semantics
default filters
time interpretation
null policy
grain
```

Compatibility can be evaluated using symbolic analysis plus a regression workload.

### Falsifiable hypothesis

Semantic-ABI compatibility classification predicts downstream query breakage substantially better than text diff, schema diff, or rule-name change heuristics.

### Killer experiment

Mutate semantic-layer definitions with controlled changes. Measure whether the ABI checker correctly predicts which historical NL questions / compiled semantic queries change answer.

### Potential paper

**Semantic Versioning for AI Data Agents: Defining Compatibility of Business Meaning.**

---

## A3. Semantic Layer CI — regression testing business meaning

Semantic layers improve Text2SQL, but they can themselves contain bugs, contradictions, incomplete branch conditions, or accidental breaking changes.

Treat the semantic layer as executable code.

A CI system generates:

- semantic regression queries;
- edge-case data instances;
- role/context variants;
- historical metric checks;
- compatibility reports.

A pull request changing one metric receives a **semantic diff**:

```text
27 historical questions unchanged
5 questions intentionally changed
2 unexpected regressions
1 previously ambiguous question became defined
```

**Hypothesis:** workload- and rule-aware semantic CI catches business-meaning regressions that ordinary dbt/data tests do not.

**Collision risk:** low-medium. Semantic layers have testing mechanisms in products, but the research problem of NL-query behavioral compatibility is substantially broader.

---

## A4. Semantic Layer Fuzzing

### Core idea

Do for business semantics what fuzzing does for programs.

Generate adversarial but realistic questions that target semantic boundary conditions:

```text
refund exactly on quarter boundary
customer changes region mid-period
order has two active status records
currency conversion date differs from invoice date
```

The goal is not to test the LLM first. The goal is to ask:

> **Is the semantic layer sufficiently specified to determine a unique correct answer?**

The fuzzer searches for question/data pairs where two reasonable interpretations remain compatible with the semantic specification.

### Scientific target

Semantic completeness, not SQL-generation accuracy.

### Killer experiment

Seed semantic models with known missing clauses. Compare random workload tests against coverage-guided semantic fuzzing on defect discovery rate.

---

## A5. Semantic Debt Index

### Problem essence

A semantic layer may exist and still leave the majority of real analyst reasoning implicit.

Define **semantic debt** as the amount of external assumption required to answer a workload correctly beyond what is encoded in the governed semantic representation.

### Possible operationalization

For each task, estimate the minimal semantic facts necessary for a correct answer. Then calculate how many are:

- explicitly encoded;
- recoverable only from raw data;
- present only in docs;
- inferred from historical behavior;
- guessed by the model.

### Metric

```text
Semantic Debt = expected unsupported semantic commitments per workload task
```

### Research value

This gives organizations and benchmarks a diagnostic more meaningful than “we have a semantic layer.”

---

## A6. PluralSemanticsSQL — there may be multiple legitimate truths

### Problem essence

A semantic layer usually assumes one canonical definition.

Real organizations do not.

`revenue` may legitimately mean different things to:

```text
finance
sales
investor reporting
product analytics
tax
```

The task is therefore not always ambiguity caused by missing information. It can be **semantic plurality**: several definitions are simultaneously authoritative in different decision contexts.

### Mechanism

Represent a semantic namespace:

```text
(term, role, purpose, jurisdiction, effective_time) -> definition
```

The agent first resolves the decision context, then compiles SQL.

### Killer benchmark

Keep the natural-language utterance nearly identical while changing only user role / decision purpose. The correct SQL intentionally changes.

### Hypothesis

Role/purpose-conditioned semantics beats a single “canonical metric” layer and generic clarification on organizations with legitimate semantic plurality.

---

# 3. Theme B — Physical schemas are representations, not the semantic object

## B1. QuotientSQL — reason over equivalence classes of schemas

### Motivation

2026 work shows that the same underlying data represented by equivalent relational schemas can cause radically different Text2SQL behavior. Additional E/R context helps but does not eliminate inconsistency.

This suggests the model is reasoning over the *representation* rather than the underlying conceptual problem.

### Mechanism

Treat semantics-preserving schema transformations as defining an equivalence relation:

```text
S1 ~ S2 ~ S3
```

The agent predicts a canonical conceptual query `C(Q)` first. Each physical schema receives a compiler:

```text
C(Q) -> SQL(S_i)
```

Training / selection enforces:

```text
C(Q | S1) = C(Q | S2) = ...
```

for schemas in the same equivalence class.

### Difference from semantic-layer agents

The semantic layer is usually hand-curated and fixed. QuotientSQL studies **representation invariance itself** and can use automatically generated schema orbits.

### Killer experiment

Use equivalent-schema generators. Compare raw SQL generation, E/R-context prompting, semantic-layer mediation, and canonical conceptual planning. Primary metric: cross-schema answer consistency at fixed ordinary accuracy.

---

## B2. Migration-Aware Query Continuity

### Problem essence

EvoSchema asks whether a model survives schema evolution. But production databases usually evolve through explicit migrations:

```text
rename column
split table
merge entities
replace source table
introduce bridge
normalize / denormalize
```

The migration itself is evidence about semantic continuity.

### Idea

Instead of regenerating SQL from scratch after every schema change, propagate a prior validated semantic plan through the migration mapping.

```text
old semantic plan
+ migration provenance
-> transformed semantic plan
-> new SQL
```

### Hypothesis

Migration-aware plan transport preserves behavior better than fresh Text2SQL regeneration on schema evolution with semantics-preserving changes.

### Research artifact

A benchmark pairing schema evolution with machine-readable migration provenance and expected query continuity.

---

## B3. Semantic Migration Certificates

For schema migrations claimed to preserve a business concept, generate a certificate that a set of semantic queries are preserved.

This changes the question from:

> “Can the LLM rewrite the SQL?”

into:

> “Can the system prove or empirically certify that the migrated representation still implements the same business meaning?”

Potential tools include relational equivalence checking, controlled database synthesis, and workload-based differential testing.

**Novelty:** database migrations are mature; combining migration correctness with NL business-query semantics is less explored.

---

## B4. JoinEntropy — hardness is ambiguity, not just hop count

### Motivation

SchemaScope shows a sharp accuracy cliff as join-hop depth rises. SchemaGraphSQL uses graph pathfinding. But two schemas with the same hop depth can have very different difficulty.

Example:

```text
A -- B -- C -- D
```

is easy if it is the only route.

A graph with eight equally plausible routes from A to D is much harder even at the same depth.

### New hardness variable

Define **join-path entropy** over plausible join paths conditioned on schema, names, FK evidence, usage and cardinality.

### Benchmark design

Factorially control:

```text
hop depth
number of alternative paths
lexical plausibility of alternatives
cardinality similarity
business-domain overlap
```

### Hypothesis

Join-path entropy explains failure variance beyond join-hop depth and schema size.

### Why this matters

It gives the field a more causal difficulty measure and suggests that the next algorithm should reduce *path ambiguity*, not merely decompose long paths.

---

## B5. Cardinality-Signature Planning

### Problem essence

Text2SQL often selects joins by names and graph connectivity, but analytical intent also implies cardinality constraints.

Examples:

```text
one row per customer
one row per invoice
exactly one current status
orders may have many lines
```

### Mechanism

Represent each candidate semantic plan with a **grain/cardinality signature** and propagate it through joins and aggregations.

Candidate join paths that cannot satisfy the requested output grain receive deterministic penalties or rejection.

### Difference from typed IR

The contribution is not a generic typed plan; it is the use of *expected cardinality behavior as a discriminative signal for join-path selection*.

### Killer experiment

Construct tasks with multiple FK-valid join routes but different fan-out behavior. Test whether cardinality signatures choose the semantically correct route.

---

# 4. Theme C — Database provenance as an agent reasoning primitive

## C1. DiffProvSQL — explain candidate disagreement with real lineage

### Problem essence

Candidate-selection systems know that SQL A and SQL B disagree, but usually do not know *why*.

DPC creates a synthetic Minimal Distinguishing Database. An orthogonal approach is to exploit **fine-grained provenance on the actual database**.

### Mechanism

For two candidate queries:

```text
Q1(D) != Q2(D)
```

compute a minimal provenance witness showing:

- output rows that differ;
- input tuples responsible;
- operators / joins that create the divergence.

The LLM sees:

```text
Candidates disagree only for customer 417.
Q1 includes invoice row I9 through join customer.account_id = invoice.account_id.
Q2 excludes it because payment P3 is not settled.
```

instead of two long SQL strings.

### Hypothesis

Minimal real-data provenance witnesses improve candidate selection and semantic debugging over raw output comparison, LLM judges, and synthetic-only distinguishing databases.

### Feasibility

ProvSQL and mature database-provenance machinery make a prototype possible for a useful SQL fragment.

---

## C2. Lineage-Directed Repair

ACTS-SQL and EXPO-SQL localize clauses / correction paths. A stronger database-grounded localization signal is **result lineage**.

If an output row is suspicious, trace it backward through joins, filters and aggregations to identify the smallest plan region responsible for its inclusion.

Repair receives:

```text
fault candidate: refund join
witness rows: ...
affected outputs: ...
unaffected plan regions: date filter, customer scope
```

### Primary metric

Fraction of already-correct semantic plan components preserved during repair.

### Novelty risk

Medium. SQL debugging and provenance are each mature, but provenance-directed LLM repair appears less occupied than clause-level model diagnosis.

---

## C3. Minimal Evidence Certificates

A high-value analytical answer should optionally carry a compact certificate:

```text
business rule versions used
schema relationships used
source query / plan
minimal lineage or aggregate witnesses
validation checks
```

The goal is not verbose explanation. It is **replayable evidence**.

### Research question

Can we compute the smallest evidence package that allows an independent verifier to reproduce or reject an agent answer?

### Metrics

- certificate size;
- verification latency;
- faithfulness;
- unsupported-claim rate;
- privacy leakage.

This connects Text2SQL with the broader 2026 push toward execution provenance in agents.

---

## C4. Provenance-Guided Clarification

When two semantic interpretations differ, not every ambiguity matters equally.

Use provenance to estimate how much of the result would change if a semantic commitment changed.

Example:

```text
"Should refunds be excluded?"
```

is worth asking if the disputed refund lineage affects 38% of the answer, but not if it affects zero rows in the current period.

### Objective

```text
clarification value = semantic uncertainty x downstream result impact
```

This is different from generic entropy-based clarification because the impact is computed from relational lineage.

---

## C5. Result Fragility Certificates

Two correct SQL queries can differ in decision robustness.

An answer may depend on:

- one anomalous row;
- one fragile join edge;
- one stale semantic rule;
- a small set of records near a threshold.

Define a **fragility score**: the smallest admissible change to data/metadata that materially changes the answer.

Potential use:

```text
"Top customer = Acme, but rank flips if one disputed invoice is removed."
```

This turns reliability from binary correctness into answer sensitivity.

---

# 5. Theme D — Databases expose optimizer statistics, but not agent statistics

## D1. AgentSynopses — semantic statistics for LLM agents

### Problem essence

A DBMS exposes statistics for the optimizer:

```text
row counts
histograms
selectivity
indexes
```

An agent needs different statistics:

```text
is this column likely a business key?
how often does this join fan out?
which enum values dominate?
is this table stale relative to alternatives?
does this look like an SCD table?
how many competing current rows exist?
which joins are frequently used in validated workloads?
```

Today the agent repeatedly discovers these facts with expensive SQL probes.

### Idea

Create an **agent-oriented synopsis layer** maintained by the database/catalog.

### Killer experiment

Equal context/token budget:

- raw schema;
- raw schema + sample rows;
- free-form probing;
- agent synopses.

Measure accuracy, DB calls, exposed rows and latency.

### Novelty

High. This is database/agent co-design rather than another prompt strategy.

---

## D2. Epistemic Catalog

Current metadata catalogs usually present facts as certain.

Instead expose:

```yaml
claim: orders.customer_id joins customers.id
source: declared_fk
confidence: 1.0
valid_from: ...
last_validated: ...

claim: invoices is the finance source of truth
source: query_log_inference
confidence: 0.78
last_validated: ...
```

An agent can reason differently about hard constraints, inferred patterns and stale documentation.

### Hypothesis

Confidence/provenance-aware metadata reduces failures in conflicting/noisy enterprise metadata without requiring more context.

### Difference from ordinary RAG

The knowledge store exposes *epistemic status*, not just text.

---

## D3. Observability Physical Design for Agents

Traditional physical design chooses indexes/materialized views to reduce query cost.

For agents, choose which **semantic observations** to precompute:

- fan-out summaries;
- join-path certificates;
- key confidence;
- freshness monitors;
- entity-value sketches;
- common business-rule checks.

Given a workload and storage budget, optimize which observations minimize future reasoning/probe cost.

### Research framing

> **Physical design for machine reasoning workloads.**

This could be a clean SIGMOD/VLDB direction.

---

## D4. Dependency-Aware Semantic Cache

Semantic caching is increasingly used to reuse analytical work, but cached SQL/answers can become invalid for many distinct reasons:

```text
schema changed
business rule changed
data snapshot changed
model-backed AI predicate changed
access policy changed
```

Store the dependency set of every cached artifact and invalidate only when a relevant dependency changes.

### Hypothesis

Dependency-aware cache invalidation preserves more safe reuse than TTL/version heuristics while sharply reducing stale semantic answers.

---

# 6. Theme E — The field needs measurement science, not only more benchmarks

## E1. Text2SQL Difficulty Geometry

### Motivation

SchemaScope isolates join-hop depth. Other work isolates schema representation, annotation quality, long context, and drift. But real failures emerge from interactions among factors.

Define a factorial difficulty space:

```text
schema size
semantic opacity
join-hop depth
join-path entropy
business-rule depth
rule drift
data sparsity
metadata conflict
output sensitivity
interaction horizon
```

### Research question

Is Text2SQL hardness approximately additive, or are there nonlinear phase boundaries where combinations cause sudden collapse?

### Artifact

A controlled generator plus response-surface analysis across models/harnesses.

### Value

The field could stop describing tasks as merely “easy / medium / hard” and start measuring *why* they are hard.

---

## E2. Benchmark Noise Decomposition

2026 annotation audits show benchmark labels can change leaderboard rankings dramatically. But annotation error is only one source of uncertainty.

Observed score varies because of:

```text
annotation uncertainty
database snapshot
model stochasticity
provider model update
prompt/harness variation
evaluation equivalence error
```

### Idea

Use hierarchical statistical modeling to decompose observed performance variance into these sources.

### Output

Instead of:

```text
Method A = 72.3
Method B = 73.1
```

report:

```text
P(B truly better than A | benchmark/model/evaluator uncertainty) = 0.61
```

### Scientific payoff

A large fraction of “SOTA” claims may be below the measurement noise floor.

---

## E3. Representation-Orbit Evaluation Tensor

Test one semantic question across multiple axes simultaneously:

```text
physical schema variant
legal database snapshot
business-rule version
SQL dialect
metadata presentation
```

A method receives a **consistency tensor**, not one execution-accuracy number.

### Core concept

True semantic understanding should be stable under changes that preserve meaning and equivariant under changes that predictably alter meaning.

This generalizes single-axis schema-robustness studies into a unified evaluation object.

---

## E4. Causal Component Leaderboards

Modern systems bundle retrieval, probing, memory, verification, ensembles and large models. Final score does not tell us which component caused the gain.

Require standardized counterfactual ablations:

```text
system
system - memory
system - probes
system - verifier
system - extra test-time compute
```

Estimate per-component causal lift and interaction effects across tasks.

### Research hypothesis

Many expensive components have positive average contribution but negative conditional contribution on identifiable task subsets.

This would complement No-One-SQL-Agent by measuring components rather than whole harnesses.

---

# 7. Theme F — Multi-step SQL agents silently assume a stationary world

## F1. SnapshotSQL — snapshot consistency for exploratory database agents

### Problem essence

SDE-SQL, PV-SQL, APEX-style agents and many tool-using systems execute multiple exploratory queries over time.

In a live database, the world can change between calls.

An agent can observe:

```text
probe 1: customer has 4 orders
probe 2: latest order is yesterday
probe 3: total revenue = X
```

where these observations never coexisted in any single database state.

The final reasoning is then based on an **impossible world** even though every SQL query individually succeeded.

### Mechanism

Bind an exploratory trajectory to a database snapshot, or explicitly track observation versions and force revalidation when they diverge.

### Benchmark

Run a controlled concurrent workload that mutates relevant tables between agent probes.

### Metrics

- impossible-world reasoning rate;
- final correctness;
- restart/revalidation cost;
- staleness tolerance.

### Novelty

High for Text2SQL/data querying. DBA-Bench includes active workloads and broader runtime diagnosis; agent transaction research addresses durable workflows, but snapshot consistency of *multi-probe analytical reasoning* is a distinct clean problem.

---

## F2. Semantic Snapshot Binding

A long-running data agent depends on more than database state:

```text
business rules
semantic layer version
schema
model endpoint
retrieval index
policy
```

Recent Semantic Isolation work identifies analogous problems in durable AI workflows.

Database agents provide a concrete domain where these resources have executable semantics.

### Direction

Bind every analytical trajectory to a **semantic snapshot manifest** and measure whether mixed-version reasoning changes SQL or answers.

### Collision risk

Medium-high because Semantic Isolation is extremely recent. A paper needs database-specific semantics and empirical failure modes, not merely a renamed isolation level.

---

## F3. Exactly-Once Intent for Text2CRUD

Network timeout after an UPDATE creates a classic ambiguity:

```text
Did the write happen or not?
```

An agent that retries blindly can double-apply a business action.

Define semantic exactly-once behavior at the user-intent level, not merely statement idempotence.

Example:

```text
"credit customer 42 with a $20 goodwill balance"
```

must have exactly one business effect despite retries, regenerated SQL, or changed execution strategy.

### Collision risk

Medium-high due to recent agentic transaction work. Most promising as a database-specific benchmark/primitive rather than a generic agent claim.

---

# 8. Theme G — Learning signals that reflect semantics rather than syntax

## G1. Metadata Causal Credit

Schema retrieval systems score relevance. But relevance is not causal usefulness.

For each metadata item, replay the same task with and without it:

```text
DDL fragment
column description
sample value
business rule
historical query
FK
```

Estimate:

```text
Delta P(correct | item)
```

Train retrieval on causal utility rather than semantic similarity.

### Hypothesis

Causal metadata credit learns smaller and more robust contexts than embedding/LLM relevance labels, especially under 84K-token enterprise contexts.

---

## G2. Semantic-Obligation RL

EXPO-SQL gives clause-level rewards. Other RL systems reward schema links, structure and execution.

A different object is a set of **semantic obligations** derived independently of SQL syntax:

```text
output grain = customer
metric = recognized_revenue
period = fiscal_Q2
join must not fan out invoices
refunds excluded after settlement date
```

Each obligation has an executable/verifiable reward.

### Difference

Two very different SQL programs can satisfy the same obligation set, so the training signal is closer to intent than clause similarity.

### Collision risk

Medium. Must demonstrate that obligations provide information unavailable from clause-level execution rewards.

---

## G3. Counterfactual Semantic Curriculum

Generate training examples where surface form and schema remain mostly unchanged while exactly one business semantic changes:

```text
cash revenue -> recognized revenue
calendar quarter -> fiscal quarter
current status -> status as of event time
```

The SQL must change accordingly.

This targets the model's tendency to memorize stable mappings instead of conditioning on current business knowledge.

### Strong evaluation

Train on one set of semantic transitions and test on unseen rule-change families.

### Collision risk

Medium because LiveSQLBench already evaluates business-rule drift; novelty must be the controlled counterfactual training methodology and generalization study.

---

# 9. New candidate programs

## Program Q1 — Temporal Semantics for Data Agents

Core thesis:

> Data changes, schemas change, and business meaning changes. Reliable agents must bind every query to the correct version of all three.

Sequence:

1. Meaning-Time benchmark.
2. Semantic ABI.
3. Semantic Layer CI.
4. Dependency-aware semantic caching.

This is stronger than generic memory-drift research because the changing objects have executable relational meaning.

## Program Q2 — Provenance-Grounded SQL Reasoning

Core thesis:

> Execution success tells an agent *what happened*; provenance tells it *why*.

Sequence:

1. DiffProvSQL candidate arbitration.
2. Lineage-directed repair.
3. Minimal evidence certificates.
4. Provenance-guided clarification.
5. Result fragility analysis.

## Program Q3 — Agent-Observable Databases

Core thesis:

> Current DBMSs expose information for human developers and query optimizers, not for autonomous semantic reasoning agents.

Sequence:

1. AgentSynopses.
2. Epistemic Catalog.
3. Observability physical design.
4. Agent workload optimizer.

This is potentially a database-systems line rather than another Text2SQL method paper.

## Program Q4 — Representation-Invariant Text2SQL

Core thesis:

> A language question refers to a conceptual data model; relational schemas are implementation choices.

Sequence:

1. JoinEntropy benchmark.
2. QuotientSQL canonical planning.
3. Migration-aware query continuity.
4. Semantic migration certificates.

## Program Q5 — Measurement Science for Text2SQL

Core thesis:

> The field cannot reliably optimize what it cannot reliably measure.

Sequence:

1. Benchmark noise decomposition.
2. Difficulty geometry.
3. Representation-orbit evaluation.
4. Causal component leaderboard.

---

# 10. Current top-12 after first collision screen

| Rank | Idea | Novelty | Feasibility | Why it is interesting now |
|---:|---|---:|---:|---|
| 1 | Meaning-Time SQL | 5 | 4 | Business-rule drift exists, but definition-time is not yet a first-class query dimension |
| 2 | DiffProvSQL | 5 | 4 | Provenance is mature DB machinery and could provide a new evidence channel for agents |
| 3 | Semantic ABI | 5 | 4 | Semantic layers are becoming infrastructure, but compatibility semantics are missing |
| 4 | AgentSynopses | 5 | 4 | Moves optimization into DB/agent co-design rather than prompt engineering |
| 5 | JoinEntropy | 5 | 5 | Cheap, falsifiable extension beyond the newly demonstrated join-hop-depth cliff |
| 6 | SnapshotSQL | 5 | 4 | Multi-probe agents assume a stationary DB that production systems do not provide |
| 7 | PluralSemanticsSQL | 5 | 4 | Challenges the one-canonical-definition assumption of semantic layers |
| 8 | QuotientSQL | 5 | 3 | Attacks representation dependence at the conceptual level |
| 9 | Semantic Layer Fuzzing | 5 | 4 | Tests whether the semantic layer itself is complete before blaming the LLM |
| 10 | Benchmark Noise Decomposition | 4 | 5 | Annotation audits suggest many reported gains may sit inside measurement uncertainty |
| 11 | Observability Physical Design | 5 | 3 | New systems problem: materialize information for reasoning agents, not only SQL optimizers |
| 12 | Semantic Debt Index | 4 | 5 | Gives an actionable measure of how much business logic remains implicit |

---

# 11. Ideas deliberately downgraded after the newest literature pass

The following should no longer receive high novelty scores without a substantially sharper twist:

- free-form SQL probing — SDE-SQL, PV-SQL, PExA;
- generic probe + verifier pipelines — PV-SQL;
- parallel atomic SQL exploration — PExA;
- tree-structured multi-strategy SQL correction — ACTS-SQL;
- clause-level error localization / RL rewards — EXPO-SQL and related work;
- long-horizon propose/execute/verify/refine training — ReEx-SQL, SQL-Trail, MTSQL-R1;
- “semantic layer improves Text2SQL” — now strongly established;
- LLM-friendly schema design — explicitly studied in 2026;
- schema-drift benchmark alone — EvoSchema and StructHallu-Drift;
- business-rule drift benchmark alone — LiveSQLBench-Large;
- generic agent transaction/isolation framework — August 2026 agentic-transaction and semantic-isolation work.

The idea pool should therefore move **one abstraction level deeper**: from agent orchestration to semantic lifecycle, database evidence, representational invariance, and systems contracts.

---

# 12. First oracle-gap / phenomenon tests before expensive engineering

A useful discipline for this expanded pool is to kill ideas cheaply.

| Direction | Cheapest decisive test |
|---|---|
| Meaning-Time SQL | Do frontier agents confuse `data_as_of` and `meaning_as_of` on controlled rule histories? |
| Semantic ABI | Do semantic-layer changes cause downstream behavioral breakage that schema/text diffs fail to predict? |
| DiffProvSQL | Given two wrong/right candidates, does minimal lineage evidence improve selection over SQL+results alone? |
| JoinEntropy | At fixed hop depth, does path entropy independently predict execution accuracy? |
| AgentSynopses | Can a small fixed synopsis replace repeated exploratory SQL without accuracy loss? |
| SnapshotSQL | Under concurrent DB updates, do multi-probe agents combine observations that never coexisted? |
| PluralSemanticsSQL | Does role/purpose context systematically change the correct SQL in realistic enterprise metrics? |
| QuotientSQL | Can a canonical conceptual plan remain stable across semantics-equivalent schema variants? |
| Semantic Fuzzing | Does coverage-guided generation find semantic-layer defects faster than random/historical workload tests? |
| Benchmark Noise | Are method deltas smaller than annotation/snapshot/model/harness uncertainty on current leaderboards? |

Only candidates with a measurable oracle/phenomenon gap should proceed to full implementation.

---

# 13. Key references for this expansion

Recent Text2SQL / data-agent frontier:

- SDE-SQL, ACL 2026: https://aclanthology.org/2026.acl-long.116/
- PV-SQL, ACL Findings 2026: https://aclanthology.org/2026.findings-acl.1286/
- PExA, ACL 2026: https://aclanthology.org/2026.acl-short.48/
- ReEx-SQL, ACL 2026: https://aclanthology.org/2026.acl-long.35/
- MTSQL-R1, ACL 2026: https://aclanthology.org/2026.acl-long.1563/
- EXPO-SQL, ACL Findings 2026: https://aclanthology.org/2026.findings-acl.1107/
- ACTS-SQL, 2026: https://arxiv.org/abs/2608.15145
- LiveSQLBench-Large / Business Rule Drift: https://livesqlbench.ai/
- EvoSchema: https://arxiv.org/abs/2603.10697
- Same Data, Different Schemas: https://arxiv.org/abs/2605.25838
- Disentangling Structure and Semantics: https://arxiv.org/abs/2608.20356
- SchemaScope, SURGeLLM 2026: https://aclanthology.org/2026.surgellm-1.17/
- Text-to-SQL Friendly Logical Database Design: https://arxiv.org/abs/2606.03145
- Semantic-Layer-Mediated Agent: https://arxiv.org/abs/2606.31041
- Semantic Layer Reliability benchmark: https://arxiv.org/abs/2604.25149
- DBA-Bench: https://arxiv.org/abs/2607.22165
- Sema: https://arxiv.org/abs/2603.11622
- Stretto: https://arxiv.org/abs/2602.04430
- Pervasive Annotation Errors: https://arxiv.org/abs/2601.08778

Adjacent mechanisms:

- database provenance / ProvSQL: https://provsql.org/
- Execution provenance survey for LLM agents: https://arxiv.org/abs/2606.04990
- Semantic Isolation for durable AI workflows: https://arxiv.org/abs/2608.05412
- Agentic Transaction: https://arxiv.org/abs/2608.13900
- STALE: https://arxiv.org/abs/2605.06527
- Supersede: https://arxiv.org/abs/2606.27472

---

# 14. Research direction after this expansion

The strongest conceptual shift is:

```text
OLD QUESTION
How do we make an LLM generate better SQL?

NEW QUESTIONS
What does the business term mean, and at what time / for which user?
What parts of that meaning are actually specified?
Which aspects of the physical schema should be irrelevant to the answer?
What database evidence explains why competing programs differ?
What observations should a DBMS expose directly to a reasoning agent?
Did the agent reason over one coherent database/semantic snapshot?
Is a reported benchmark improvement larger than the measurement uncertainty?
```

That changes Text2SQL from a narrow program-generation task into a research testbed for **semantic systems, evidence-grounded agents, and AI-native data infrastructure**.