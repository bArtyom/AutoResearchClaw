# Text2SQL × Agent Research Map (2026)

> Date: 2026-08-21  
> Scope: research opportunities at the intersection of Text-to-SQL, tool-using agents, interactive database assistants, and AutoResearchClaw-style autonomous research.  
> Status: exploratory research memo, intended to seed experiments and implementation work.

## 1. Why this is no longer "just Text2SQL"

Classical Text2SQL is often framed as a one-shot semantic parsing problem:

`natural-language question -> SQL`

That framing is increasingly too narrow for realistic enterprise data work. Modern benchmarks are moving toward workflows in which a system must inspect a large schema, read documentation, ask clarifying questions, call tools, execute queries, diagnose failures, refine SQL, maintain conversational state, and sometimes modify database state.

Two signals are especially important:

- **Spider 2.0** explicitly targets real-world enterprise Text2SQL workflows, including very large schemas, multiple SQL dialects, and code-agent / DBT-style tasks rather than only single-query generation.
- **BIRD-Interact** evaluates dynamic interaction in both conversational and agentic modes, with database documentation, a user simulator, full CRUD-style behavior, executable tests, and long multi-turn trajectories.

This suggests a useful reframing:

> The frontier problem is not "generate SQL"; it is **build an evidence-grounded database reasoning agent whose actions happen to include SQL**.

That shift opens a much larger research space.

## 2. Research landscape

### 2.1 One-shot Text2SQL

Core subproblems:

- schema linking
- value grounding
- join-path inference
- SQL composition
- nested-query reasoning
- aggregation / grouping semantics
- dialect adaptation
- execution correctness

This remains a strong baseline and is still useful as a controlled setting for ablations.

### 2.2 Execution-guided Text2SQL

The model generates SQL, executes it, observes errors or result statistics, then repairs the query.

Research questions:

- Which execution signals are most useful: parser error, DB error, row count, column types, result samples, query plan, constraint violations?
- How many repair rounds are optimal before cost grows faster than accuracy?
- Can a learned repair policy decide whether to edit locally or re-plan globally?
- How can one distinguish a syntactically valid but semantically wrong query from a correct query?

### 2.3 Agentic Text2SQL

The model gets tools rather than a single prompt. Candidate tools:

- `list_tables`
- `describe_table`
- `search_schema`
- `sample_rows`
- `execute_sql`
- `explain_query_plan`
- `search_docs`
- `inspect_foreign_keys`
- `ask_user`
- `compare_candidate_queries`
- `rollback_transaction`

The scientific question becomes an agent-design problem: **what policy should decide which tool to call next?**

### 2.4 Interactive Text2SQL

A realistic request is often underspecified:

> "Show me our best customers last quarter."

"Best" may mean revenue, margin, retention, LTV, order count, or strategic account tier. An agent that silently assumes one interpretation can produce perfectly executable but operationally wrong SQL.

Research opportunities:

- uncertainty-aware clarification
- value-of-information estimation
- minimal-question strategies
- active disambiguation under turn budgets
- user adaptation: novice vs analyst vs database engineer
- learning when *not* to ask a question

### 2.5 Multi-agent SQL reasoning

Instead of one model doing everything, specialize roles:

1. **Intent Agent** — formalizes the business question and ambiguity.
2. **Schema Agent** — retrieves candidate tables, columns, join paths, semantic metadata.
3. **Planner Agent** — decomposes the task into relational operations.
4. **SQL Writer** — materializes SQL for a target dialect.
5. **Executor Agent** — runs SQL and captures structured evidence.
6. **Critic Agent** — looks for semantic, cardinality, leakage, and aggregation errors.
7. **Verifier Agent** — performs counterfactual / metamorphic checks.
8. **Clarifier Agent** — decides whether a user question is worth asking.

A key research question is whether role decomposition actually improves reliability after controlling for total token budget and number of model calls.

## 3. High-value research ideas

## Idea A — Budgeted Active Schema Discovery

### Thesis

Large enterprise schemas make dumping the full DDL into the prompt both expensive and noisy. Treat schema linking as **active information acquisition**.

### Agent loop

1. Read user question.
2. Predict a distribution over likely schema regions.
3. Call `search_schema` / `describe_table` selectively.
4. Update belief state.
5. Stop when marginal expected value of another schema call falls below its cost.
6. Generate SQL.

### Novelty angle

Most systems retrieve schema context, but fewer explicitly optimize **accuracy vs schema-inspection cost** as a sequential decision problem.

### Metrics

- execution accuracy
- schema recall@k
- schema tokens consumed
- number of metadata tool calls
- wall-clock latency
- dollars per solved task

### Strong ablation

Compare:

- full-schema prompting
- top-k embedding retrieval
- heuristic iterative retrieval
- learned / LLM value-of-information policy

## Idea B — Semantic Verification via Metamorphic SQL Tests

### Thesis

Execution success is weak evidence. A wrong SQL statement can return plausible rows. Add **metamorphic tests** that should preserve or predictably alter results.

### Examples

- Rewrite an inner query as an equivalent CTE and compare results.
- Add a logically redundant predicate and test invariance.
- Replace `COUNT(*)` with a derived equivalent when keys are known.
- Perturb date boundaries and check monotonicity.
- Compare aggregation before and after explicit deduplication when cardinality assumptions matter.
- Sample a few output rows and verify the implied join path manually through targeted subqueries.

### Agent design

`SQL candidate -> generate semantic invariants -> execute tests -> score evidence -> accept / repair`

### Research question

Can metamorphic verification reduce **silent semantic errors** without access to gold SQL?

This is attractive because it turns database execution itself into a verifier.

## Idea C — Counterexample-Guided SQL Repair

Borrow the flavor of CEGIS (counterexample-guided inductive synthesis).

1. Generate SQL.
2. Run it.
3. Produce a semantic claim about what the result must satisfy.
4. Search the database for a counterexample.
5. If found, explain which assumption failed.
6. Patch the plan / SQL.

Example:

> Query claims "one row per customer" but an auxiliary check finds duplicate customer IDs.

That counterexample can reveal an incorrect many-to-many join even when the main result looks reasonable.

## Idea D — Ask-or-Act: Clarification as a Learned Policy

### Thesis

The key agent decision is often not which SQL to write but whether to ask the user first.

Define actions:

- `ASK(question)`
- `INSPECT(tool_call)`
- `EXECUTE(sql)`
- `ANSWER(result)`

Optimize a reward combining:

- task correctness
- number of user interruptions
- total latency
- token / query cost
- unsafe side-effect penalty

Research variants:

- entropy threshold
- self-consistency disagreement
- explicit expected value of information
- learned classifier / policy model
- bandit policy personalized to user behavior

## Idea E — SQL Agent with a Typed Intermediate Representation

Direct NL -> SQL generation makes repair brittle. Introduce a typed relational IR:

```text
Goal
  -> Entities
  -> Filters
  -> Time window
  -> Measures
  -> Dimensions
  -> Join graph
  -> Aggregation semantics
  -> Ordering / limit
  -> SQL dialect lowering
```

Benefits:

- easier debugging
- dialect portability
- stronger static checks
- clearer separation between semantic planning and syntax
- localized repair

Research hypothesis:

> A typed relational-plan IR improves cross-dialect transfer and semantic repair more than chain-of-thought-only planning under equal inference cost.

## Idea F — Multi-Agent Debate, But Over Structured Disagreements

Generic "agents debate each other" often adds tokens without adding information. Make the disagreement space explicit.

Each candidate must output:

- chosen tables
- chosen join keys
- grain of each intermediate relation
- aggregation grain
- filter interpretation
- assumptions
- final SQL

A judge compares these structured decisions, not prose.

Potential selection signals:

- majority agreement on join graph
- execution result consistency
- invariant checks
- cardinality sanity checks
- model confidence calibration

## Idea G — Query-Plan-Aware Text2SQL Agent

Expose `EXPLAIN` / query plan information to the agent.

Two objectives:

1. semantic correctness
2. operational efficiency

A production-grade agent should avoid queries that scan terabytes or accidentally create Cartesian products.

Research questions:

- Can the agent predict expensive or pathological plans before execution?
- Does query-plan feedback help detect wrong joins?
- Can one jointly optimize correctness and estimated compute cost?

This creates a bridge from Text2SQL to **Text2OptimizedSQL**.

## Idea H — Read/Write Safety for Database Agents

As benchmarks move into CRUD, database agents become action agents.

Recommended research architecture:

- SQL AST policy layer
- read-only default
- explicit transaction boundary
- dry-run mode
- row-count impact prediction
- destructive-operation confirmation
- automatic rollback in evaluation
- policy-based table / column access control

Potential paper framing:

> Constrained Agentic Text2SQL: Maximizing Task Success Under Database Safety Policies.

## Idea I — Persistent SQL Memory and Failure Reuse

AutoResearchClaw already has a self-evolution motif. A SQL agent can maintain memory at several levels:

- schema aliases and business glossary
- previously validated join paths
- reusable query fragments
- error -> repair patterns
- dialect-specific pitfalls
- user-specific definitions (e.g. "active customer")

Important research distinction:

- retrieval of successful examples
- retrieval of *failures and repairs*
- schema-specific memory vs transferable memory

Hypothesis:

> Failure-conditioned memory yields larger gains on recurring enterprise schemas than success-example retrieval alone.

## Idea J — Autonomous Research Agent for Text2SQL Itself

This is the most direct connection to AutoResearchClaw.

Give the system a research question such as:

> "Does uncertainty-triggered clarification improve BIRD-Interact success per dollar?"

The autonomous research loop could:

1. retrieve relevant papers / repos
2. formulate hypotheses
3. implement competing agent policies
4. run benchmark subsets
5. analyze cost / correctness tradeoffs
6. inspect failure clusters
7. propose the next experiment
8. write the paper

This makes Text2SQL not only an application domain but also a **testbed for autonomous agent research**.

## 4. Proposed flagship project: SQL ResearchClaw

A strong umbrella project would combine the ideas above into one architecture.

```text
User Task
   |
   v
Intent & Ambiguity Analyzer
   |------> ask_user (optional)
   v
Active Schema Explorer
   |
   v
Relational Plan IR
   |
   +----> Candidate SQL Generator A
   +----> Candidate SQL Generator B
   +----> Candidate SQL Generator C
                 |
                 v
          Safe SQL Executor
                 |
                 v
       Semantic Verifier / Critic
        |     |       |       |
        |     |       |       +--> EXPLAIN / cost checks
        |     |       +----------> metamorphic tests
        |     +------------------> counterexample probes
        +------------------------> result sanity checks
                 |
          accept / repair / ask
                 |
                 v
              Answer
                 |
                 v
      Failure & Repair Memory
```

### Key principle

Do not reward the agent for producing SQL. Reward it for producing **verified task outcomes**.

## 5. Benchmark strategy

A credible research program should span multiple regimes.

### Regime 1 — Fast local iteration

Use small SQLite / DuckDB tasks for:

- planner ablations
- repair-loop development
- metamorphic-test design
- cost instrumentation

### Regime 2 — Classical Text2SQL

Use established single-turn benchmarks to isolate semantic parsing performance.

### Regime 3 — Enterprise workflow

Use Spider 2.0-style tasks to stress:

- very large schemas
- multiple dialects
- workflow / repository context
- tool usage

### Regime 4 — Interactive agent

Use BIRD-Interact / Mini-Interact-style tasks to measure:

- clarification
- multi-turn planning
- stateful DB actions
- user interaction
- agent efficiency

## 6. Metrics beyond exact / execution match

For agentic research, report a richer vector:

- task success / executable test pass rate
- execution accuracy
- semantic error rate on executable-but-wrong queries
- number of DB calls
- number of schema calls
- user turns
- tokens
- dollar cost
- latency
- unsafe-query rate
- rollback rate
- query-plan cost / bytes scanned where available
- repair success conditioned on first-attempt failure
- calibration: confidence vs correctness

A useful scalar for some experiments is:

```text
utility = success_reward
          - lambda_db * db_calls
          - lambda_tok * tokens
          - lambda_user * clarification_turns
          - lambda_risk * unsafe_actions
```

But always publish the raw components too.

## 7. Failure taxonomy to instrument from day one

Each failed episode should be classified automatically when possible:

1. intent misunderstanding
2. unresolved ambiguity
3. schema retrieval miss
4. wrong table
5. wrong column
6. wrong join key
7. wrong join cardinality
8. missing / wrong filter
9. wrong temporal interpretation
10. wrong aggregation grain
11. wrong grouping
12. wrong ordering / top-k
13. dialect syntax error
14. execution/runtime error
15. semantic silent error
16. state-management error
17. unsafe mutation
18. unnecessary clarification
19. excessive tool use
20. verifier false positive / false negative

This taxonomy is itself valuable: it supports targeted repair agents and produces interpretable research results.

## 8. Most promising paper-sized hypotheses

### P1 — Value-of-information schema agent

**Hypothesis:** active schema exploration reaches equal or higher execution accuracy than top-k schema retrieval with materially fewer schema tokens on large schemas.

### P2 — Metamorphic verifier

**Hypothesis:** execution + metamorphic verification reduces silent semantic errors compared with execution-guided self-correction alone.

### P3 — Ask-or-act policy

**Hypothesis:** uncertainty-triggered clarification improves interactive success per dollar over always-ask and never-ask baselines.

### P4 — Structured multi-agent consensus

**Hypothesis:** consensus over join graph + relation grain + aggregation semantics outperforms free-form multi-agent debate under the same token budget.

### P5 — Failure-memory SQL agent

**Hypothesis:** retrieving prior failure-repair traces improves recurring-schema performance more than retrieving only successful exemplars.

### P6 — Plan-aware SQL optimization

**Hypothesis:** adding `EXPLAIN` feedback reduces both expensive queries and join-related semantic errors without hurting task success.

## 9. What I would build first

A pragmatic first implementation should avoid a giant multi-agent system.

### V0

- SQLite / DuckDB backend
- schema search tool
- table describe tool
- SQL execute tool
- SQL AST safety checker
- single planner/writer agent
- execution repair loop
- structured trajectory logging

### V1

Add:

- typed relational IR
- semantic critic
- metamorphic verifier
- token / DB-call budget

### V2

Add:

- active clarification
- multi-candidate generation
- structured consensus
- persistent failure memory

### V3

Add:

- PostgreSQL / Snowflake / BigQuery adapters
- BIRD-Interact integration
- Spider 2.0 integration
- autonomous experiment sweeps through AutoResearchClaw

## 10. Relationship to AutoResearchClaw

AutoResearchClaw is a particularly good host because its architecture already separates a domain-agnostic research pipeline from domain-specific profiles, prompt adapters, experiment sandboxes, and benchmark manifests.

A Text2SQL integration can therefore exist at two levels:

### Level A — Text2SQL as a research domain

Add a `text2sql` domain profile and prompts so the 23-stage pipeline can autonomously research Text2SQL algorithms.

### Level B — SQLAgent as a specialist experiment executor

Add a dedicated SQL experiment sandbox that can launch benchmark runs, provision databases, call model agents, collect traces, and output machine-readable metrics.

The combination is more powerful than either alone:

> AutoResearchClaw proposes and evaluates new SQL-agent algorithms; SQLAgent provides the executable laboratory.

## 11. External references

- Spider 2.0 project: https://spider2-sql.github.io/
- Spider 2.0 repository: https://github.com/xlang-ai/Spider2
- BIRD-Interact project: https://bird-interact.github.io/
- BIRD-Interact repository: https://github.com/bird-bench/BIRD-Interact

These benchmarks evolve over time; benchmark versions, data snapshots, evaluation images, and leaderboard dates should be pinned in any reproducible experiment.
