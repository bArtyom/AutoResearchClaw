# Text2SQL × AutoResearchClaw Integration Plan

> Date: 2026-08-21  
> Goal: turn AutoResearchClaw into an autonomous laboratory for Text2SQL and database-agent research, while keeping the core 23-stage pipeline domain-agnostic.

## 1. Architectural fit

AutoResearchClaw already exposes the right extension points for this project:

- domain profile
- prompt adapter / optional prompt bank
- experiment mode
- sandbox executor
- benchmark manifests + rubrics
- self-evolution / failure memory
- human-in-the-loop intervention

The existing domain integration guide is explicit that the pipeline runner itself should remain generic. Text2SQL should therefore be added as a domain plugin plus a specialist experiment executor rather than as hard-coded SQL logic in the main stages.

## 2. Proposed domain id

Use:

```text
text2sql
```

Potential future subdomains:

```text
text2sql_classic
text2sql_enterprise
text2sql_interactive
sql_agent
```

Start with one `text2sql` profile and use experiment config to select benchmark / task regime. Split only if prompt behavior becomes materially different.

## 3. Proposed files

Minimum research-domain integration:

```text
researchclaw/domains/profiles/text2sql.yaml
researchclaw/domains/adapters/text2sql.py
researchclaw/domains/prompt_adapter.py               # registry entry
researchclaw/domains/detector.py                     # keyword rules
```

Recommended full integration:

```text
researchclaw/prompts/text2sql.py
researchclaw/prompts/manager.py                      # bank registration
researchclaw/experiment/sql_agent_sandbox.py
researchclaw/experiment/factory.py                   # mode dispatch
researchclaw/config.py                               # SQLAgentConfig + mode
experiments/arc_bench/config/text2sql/topics.yaml
experiments/arc_bench/config/text2sql/manifests/
experiments/arc_bench/config/text2sql/rubrics/
```

Optional reusable package:

```text
researchclaw/sql_agent/
    __init__.py
    tools.py
    schema_index.py
    planner.py
    executor.py
    verifier.py
    safety.py
    memory.py
    trajectory.py
    dialects/
```

## 4. Domain profile draft

Suggested profile semantics:

```yaml
domain_id: text2sql
display_name: Text-to-SQL and Database Agents
parent_domain: computing

preferred_experiment_mode: sql_agent
preferred_project_mode: full-auto
preferred_target_conference: neurips
default_time_budget_sec: 3600
default_max_iterations: 5
default_metric_key: task_success
default_metric_direction: maximize

experiment_paradigm: benchmark

condition_terminology:
  baseline: baseline Text2SQL / SQL-agent system
  proposed: proposed SQL reasoning or agent method
  variant: ablation / policy variant
  input: natural-language database task + schema + documentation
  metric: task success / execution accuracy / cost-adjusted success

core_libraries:
  - sqlglot
  - sqlalchemy
  - pandas
  - duckdb

metric_types:
  - scalar
  - table
  - structured

statistical_tests:
  - paired_bootstrap
  - mcnemar
  - wilcoxon_signed_rank

output_formats:
  - latex_table
  - failure_breakdown
  - cost_accuracy_frontier

figure_types:
  - success_vs_cost
  - failure_taxonomy_bar
  - repair_curve
  - tool_call_distribution

code_generation_hints: |
  Text2SQL experiment requirements:
  1. Separate semantic planning from SQL execution.
  2. Log every tool call, SQL candidate, DB response, and repair step.
  3. Pin benchmark version and database snapshot.
  4. Never mutate benchmark databases unless the benchmark explicitly requires CRUD.
  5. Run destructive operations inside rollbackable transactions.
  6. Record tokens, latency, DB calls, and user turns in addition to accuracy.
  7. Emit results.json with per-task traces and aggregate metrics.
```

## 5. SQL Agent sandbox

### 5.1 Sandbox responsibilities

The sandbox should be an **evaluation harness**, not the intelligence itself.

It should:

1. provision / connect to the benchmark DB
2. expose safe database tools
3. launch an agent policy under test
4. enforce query / token / wall-clock budgets
5. capture trajectories
6. compute benchmark metrics
7. generate failure summaries
8. emit a stable `results.json`

### 5.2 Suggested interface

```python
class SQLAgentSandbox(SandboxProtocol):
    def prepare(self, run_dir, config): ...
    def execute(self, experiment_spec): ...
    def collect_results(self): ...
    def cleanup(self): ...
```

Config sketch:

```yaml
experiment:
  mode: sql_agent
  sql_agent:
    benchmark: mini_interact
    dialect: sqlite
    task_limit: 50
    max_agent_turns: 20
    max_db_calls: 30
    max_user_turns: 5
    read_only: true
    allow_explain: true
    allow_sample_rows: true
    save_trajectories: true
```

## 6. Tool API

A constrained tool interface makes experiments reproducible and simplifies ablations.

### Metadata tools

```text
list_tables()
describe_table(table)
search_schema(query, k)
get_foreign_keys(table)
search_docs(query, k)
```

### Data inspection tools

```text
sample_rows(table, columns, limit)
get_distinct_values(table, column, limit)
get_column_stats(table, column)
```

### Execution tools

```text
execute_sql(sql)
explain_sql(sql)
validate_sql(sql)
```

### Interaction tools

```text
ask_user(question)
```

### Safety / state tools

```text
begin_transaction()
rollback_transaction()
commit_transaction()  # disabled by default in evaluation
```

Important: tool results should be structured JSON, not free-form strings whenever possible.

## 7. Structured trajectory schema

Every benchmark episode should write a trace such as:

```json
{
  "task_id": "...",
  "benchmark": "...",
  "model": "...",
  "question": "...",
  "steps": [
    {
      "turn": 1,
      "action": "search_schema",
      "args": {"query": "customer revenue"},
      "observation": {},
      "latency_ms": 120
    },
    {
      "turn": 2,
      "action": "execute_sql",
      "args": {"sql": "SELECT ..."},
      "observation": {
        "status": "ok",
        "row_count": 10
      }
    }
  ],
  "final_sql": "...",
  "success": true,
  "failure_type": null,
  "metrics": {
    "tokens": 8120,
    "db_calls": 8,
    "schema_calls": 3,
    "user_turns": 1,
    "wall_clock_sec": 12.3
  }
}
```

This trace format is critical because many interesting research questions concern **behavior**, not just final accuracy.

## 8. Verifier stack

A SQL research agent should support pluggable verifier modules.

### V0 — Syntax / execution verifier

Checks:

- parseability
- database execution
- runtime errors

### V1 — Static semantic checks

Checks:

- referenced tables / columns exist
- suspicious Cartesian joins
- grouping consistency
- aggregate/non-aggregate misuse
- destructive statements

Use an AST library such as `sqlglot` to normalize across dialects.

### V2 — Cardinality verifier

The agent records intended grain:

```text
one row per customer
one row per month
one row per product-category
```

The verifier runs probes to test whether the result actually satisfies that grain.

### V3 — Metamorphic verifier

Generate invariants / equivalent rewrites and compare results.

### V4 — Counterexample verifier

Search for rows that violate the agent's semantic assumptions.

Ablating V0..V4 gives a clean paper experiment.

## 9. Relational-plan IR

Proposed typed representation:

```yaml
intent:
  question: "..."
  ambiguities: []

entities:
  - customer
  - order

measures:
  - expression: revenue
    aggregation: sum

dimensions:
  - customer_id

filters:
  - field: order_date
    operator: between
    value: last_quarter

join_graph:
  - left: customers.customer_id
    right: orders.customer_id
    cardinality: one_to_many

result_grain:
  keys:
    - customer_id

ordering:
  - expression: revenue
    direction: desc

limit: 10
```

Research value:

- easier planner/writer separation
- explicit semantics for critic agents
- cross-dialect SQL lowering
- structured disagreement between candidates
- deterministic static checks

## 10. Safety design

The sandbox must make unsafe behavior measurable and containable.

### Defaults

- read-only connection where possible
- reject DDL / DML for read-only tasks
- cap result rows
- cap execution time
- reject multiple statements unless required
- detect `CROSS JOIN` / missing predicates as warnings
- use `EXPLAIN` / dry run before expensive warehouse queries

### CRUD benchmarks

For write tasks:

- isolated database per task
- transaction boundary
- snapshot / restore
- automatic rollback after scoring
- explicit affected-row logging

Safety should become a metric, not merely an implementation detail.

## 11. Benchmark ladder

### Stage A — local synthetic / unit benchmark

Purpose: fast iteration.

- SQLite
- DuckDB
- generated schemas
- controlled ambiguity
- known join cardinalities

### Stage B — classical Text2SQL subset

Purpose: compare against one-shot baselines and isolate semantic parsing.

### Stage C — Spider 2.0

Purpose: enterprise-scale schemas, dialects, workflow context, repository / DBT tasks.

### Stage D — BIRD-Interact / Mini-Interact

Purpose: interactive behavior, clarification, user simulator, CRUD / BI tasks, agentic control.

Benchmark versions must be pinned. Do not publish only a benchmark name; record commit / dataset snapshot / Docker image hash where feasible.

## 12. ARC-Bench extension proposal

Create a new ARC-Bench family with 8-12 topics.

Example topics:

### SQL01 — Active schema retrieval

Question:

> Does iterative value-of-information schema discovery improve task success per schema token versus static top-k retrieval?

Conditions:

- full schema
- embedding top-k
- iterative heuristic retrieval
- uncertainty/value-of-information retrieval

Metrics:

- execution accuracy
- schema token count
- metadata calls
- latency

### SQL02 — Execution repair

Question:

> Which feedback signals best improve repair after first-attempt SQL failure?

Conditions:

- no repair
- DB error only
- error + result sample
- error + result stats
- error + plan / EXPLAIN

### SQL03 — Metamorphic verification

Question:

> Can metamorphic SQL checks detect executable-but-wrong queries?

### SQL04 — Clarification policy

Question:

> When should an interactive SQL agent ask the user rather than act?

### SQL05 — Structured multi-agent consensus

Question:

> Does consensus over join graph / result grain beat free-form debate at equal token budget?

### SQL06 — Failure memory

Question:

> Does retrieval of prior failures + repairs outperform successful-example retrieval on recurring schemas?

### SQL07 — Query-plan-aware agent

Question:

> Does `EXPLAIN` feedback reduce cost and semantic join errors?

### SQL08 — Typed relational IR

Question:

> Does plan-first generation improve cross-dialect transfer and repair locality?

### SQL09 — Safety-constrained CRUD agent

Question:

> How much success is lost when enforcing strict safety policies, and which policies provide the best risk-success frontier?

### SQL10 — Budget allocation

Question:

> Under a fixed inference budget, should extra tokens go to schema exploration, candidate generation, or verification?

## 13. Example rubric dimensions

Each ARC-Bench Text2SQL topic can use rubric axes like:

```text
25% task correctness
20% experimental rigor
15% cost / efficiency analysis
15% failure analysis
10% statistical analysis
10% reproducibility
 5% safety / operational constraints
```

For agent topics, require trajectory-level evidence rather than accepting only an aggregate score.

## 14. Experimental methodology requirements

For all proposed-agent papers:

### Control total compute

Multi-agent methods often look better simply because they call the LLM more times. Report:

- total prompt tokens
- total completion tokens
- number of LLM calls
- number of DB calls
- wall-clock latency
- estimated dollar cost

When claiming an architectural advantage, include at least one equal-budget comparison.

### Paired evaluation

Run methods on identical tasks / database snapshots. Prefer paired tests and confidence intervals.

### Separate first-pass vs repair gains

Report:

```text
first_attempt_success
final_success_after_repair
repair_success_given_initial_failure
```

### Failure-aware reporting

Always include error category shifts. A method that reduces syntax errors but increases silent semantic errors may be worse despite a similar headline metric.

## 15. AutoResearchClaw closed-loop research concept

The long-term goal is a loop like:

```text
Research question
    ↓
Literature / prior agent retrieval
    ↓
Hypothesis generation
    ↓
Agent architecture synthesis
    ↓
Code generation
    ↓
SQL benchmark sandbox
    ↓
Trajectory + metric collection
    ↓
Failure clustering
    ↓
New hypothesis / repair operator
    ↓
Next experiment
    ↓
Paper writing + review
```

This is unusually well aligned with Text2SQL because benchmark outcomes are highly executable and machine-verifiable.

## 16. Evolution / memory integration

AutoResearchClaw's self-evolution can store SQL-specific lessons in a structured way.

Suggested memory record:

```yaml
scope:
  dialect: snowflake
  benchmark: spider2
  schema_pattern: many_to_many_bridge

failure:
  category: wrong_join_cardinality
  symptom: duplicate entity rows after aggregation

repair:
  strategy: aggregate bridge relation before joining

validation:
  tasks_improved: 7
  tasks_regressed: 1
  confidence: 0.84
```

Only promote lessons that survive cross-task validation. Otherwise the system may memorize task-specific artifacts.

## 17. Milestones

### M0 — Research scaffolding

Deliverables:

- `text2sql` domain profile
- prompt adapter
- detector rules
- research docs

### M1 — Local SQL sandbox

Deliverables:

- SQLite/DuckDB tool API
- safe executor
- trajectory logger
- baseline planner + repair loop
- 20-50 local tasks

### M2 — Verification research

Deliverables:

- typed IR
- cardinality checks
- metamorphic tests
- verifier ablation study

### M3 — Interactive research

Deliverables:

- `ask_user` action
- uncertainty policy
- Mini-Interact adapter
- ask-vs-act experiments

### M4 — Enterprise benchmarks

Deliverables:

- Spider 2.0 adapter
- Snowflake/BigQuery guardrails
- query-plan instrumentation

### M5 — Autonomous paper loop

Deliverables:

- ARC-Bench Text2SQL topics
- automated experiment sweeps
- failure-to-next-hypothesis feedback
- end-to-end generated research report

## 18. Initial implementation priority

If engineering time is limited, prioritize in this order:

1. stable trajectory schema
2. safe execution tools
3. single-agent baseline
4. typed relational IR
5. verification stack
6. cost instrumentation
7. interactive clarification
8. multi-agent specialization
9. persistent memory

This order matters. Without good traces, execution control, and reproducible metrics, a multi-agent architecture is difficult to study scientifically.

## 19. Definition of success

The integration should eventually support a command conceptually equivalent to:

```bash
researchclaw run \
  --profile text2sql \
  --topic "Does metamorphic verification reduce silent semantic errors in agentic Text2SQL?" \
  --auto-approve
```

and autonomously produce:

- literature review
- research hypothesis
- proposed SQL-agent method
- generated implementation
- benchmark configuration
- benchmark trajectories
- statistical analysis
- cost / failure breakdowns
- paper draft
- reproducibility artifacts

That would turn AutoResearchClaw from a generic autonomous research pipeline into a **self-improving research system for database agents**.

## 20. References / benchmark entry points

- AutoResearchClaw domain integration guide: `docs/DOMAIN_INTEGRATION_GUIDE.md`
- ARC-Bench: `experiments/arc_bench/`
- Spider 2.0: https://spider2-sql.github.io/
- Spider 2.0 repo: https://github.com/xlang-ai/Spider2
- BIRD-Interact: https://bird-interact.github.io/
- BIRD-Interact repo: https://github.com/bird-bench/BIRD-Interact
