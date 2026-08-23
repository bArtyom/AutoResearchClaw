# CertaintySQL / Fault-or-Data? — Feasibility, Novelty, and Experiment Deep Dive (2026)

> Date: 2026-08-23  
> Status: literature-driven research spike; no runtime implementation changes.  
> Goal: determine whether `CertaintySQL` is a genuinely publishable Text2SQL/database-agent direction, identify the tractable technical core, and reduce it to a falsifiable experiment specification suitable for AutoResearchClaw.

---

## 1. Executive conclusion

After a deeper literature audit, the broad idea **survives**, but the contribution must be phrased carefully.

The novel claim should **not** be:

> “Use database repairs / consistent query answering with an LLM.”

Database repair and Consistent Query Answering (CQA) are mature research areas with strong theory and systems. CAvSAT already handles unions of conjunctive queries and denial constraints using SAT/Weighted MaxSAT; LinCQA provides SQL/Datalog rewritings with linear-time guarantees for a tractable class of acyclic SPJ queries; recent work extends range-consistent answers to aggregation; operational CQA even provides probabilistic/approximate semantics.

The stronger and more defensible Text2SQL contribution is:

> **Current SQL agents conflate program faults with data faults. We introduce a controlled diagnostic task in which the agent must distinguish QUERY_FAULT, DATA_FAULT, BOTH, and INSUFFICIENT_EVIDENCE, and must avoid “repairing” a semantically correct SQL query merely because the underlying database violates integrity assumptions.**

CQA then becomes a principled evidence engine inside that diagnostic task rather than the novelty claim by itself.

A suitable paper name is:

> **Fault-or-Data? Diagnosing Query Errors under Inconsistent Databases for Reliable Text-to-SQL Agents**

`CertaintySQL` can remain the name of the answer-certification component.

---

# 2. Closest prior work and why the gap still exists

## 2.1 Consistent Query Answering is mature

The classical CQA framework assumes a database can violate integrity constraints. Rather than choosing one cleaned database, it considers **repairs**: minimally changed database instances satisfying the constraints. An answer is *consistent/certain* when it holds in every repair.

Key references:

- Bertossi, *Database Repairing and Consistent Query Answering*, 2011.
- Dixit & Kolaitis, *A SAT-based System for Consistent Query Answering*, 2019. https://arxiv.org/abs/1905.02828
- Fan et al., *LinCQA: Faster Consistent Query Answering with Linear Time Guarantees*, 2022. https://arxiv.org/abs/2208.12339
- Figueira et al., *A Simple Algorithm for Consistent Query Answering Under Primary Keys*, ICDT 2023 / LMCS 2025.
- Koutris, Ouyang & Wijsen, work on path/rooted-tree queries under primary keys, 2023–2024.

Therefore, a database paper claiming only “we answer queries over inconsistent databases” would have no novelty.

## 2.2 Aggregation is no longer an obvious open gap

CAvSAT-style work has already handled `COUNT`, `SUM`, `MIN`, `MAX`, and grouping using SAT-based techniques.

More recent theory gives **range-consistent answers** to aggregation queries: instead of pretending there is one exact numerical answer, report the minimum and maximum obtainable across repairs.

References:

- Dixit & Kolaitis, *Consistent Answers of Aggregation Queries using SAT Solvers*, 2021. https://arxiv.org/abs/2103.03314
- Amezian El Khalfioui & Wijsen, *Computing Range Consistent Answers to Aggregation Queries via Rewriting*, 2024. https://arxiv.org/abs/2409.01648
- Amezian El Khalfioui & Wijsen, *Computing Consistent Least Upper Bounds in Aggregate Logic*, ICDT 2026.

For a numerical answer, the useful interface is therefore naturally:

```text
ordinary_answer = 12,481
range_consistent_answer = [12,102, 12,577]
```

or, if the range collapses:

```text
range_consistent_answer = [12,481, 12,481]
status = CERTAIN
```

Again, the novelty is not the range itself; it is using this signal to change the behavior of an NL-to-SQL agent.

## 2.3 Recent operational CQA raises the bar further

Operational CQA constructs repairs by sequences of operations such as tuple deletions and can assign probabilities over operational repairs / repair sequences. This supports approximate answer probabilities with explicit guarantees in useful fragments.

References:

- Calautti et al., *Uniform Operational Consistent Query Answering*, PODS 2022; TODS 2026. https://arxiv.org/abs/2204.10592
- Calautti et al., *Combined Approximations for Uniform Operational Consistent Query Answering*, 2025. https://arxiv.org/abs/2508.15814

This means even “assign a confidence based on how many repairs support the answer” is not novel in database theory.

For `Fault-or-Data?`, operational CQA is better treated as an optional diagnostic feature:

```text
P(answer survives repair process) = 0.91
```

rather than the core theoretical contribution.

## 2.4 Text2SQL robustness work also narrows the gap

`SynSQL` (2026) synthesizes alternative, schema-consistent databases conditioned on a question to stress-test Text2SQL systems and exposes errors hidden by a single static database.

Reference:

- Habibollah & Rafiei, *SynSQL: Synthesizing Relational Databases for Robust Evaluation of Text-to-SQL Systems*, 2026. https://arxiv.org/abs/2604.27261

Therefore, our benchmark must **not** be framed simply as “evaluate SQL on multiple database instances.”

The distinction is:

```text
SynSQL:
    alternative schema-consistent worlds
    -> expose query semantic errors hidden by one snapshot

Fault-or-Data?:
    explicitly inconsistent world + known integrity assumptions
    -> decide whether abnormal behavior originates in the SQL,
       the data, both, or cannot be determined
```

These are complementary.

## 2.5 Benchmark annotation errors are a separate axis

VLDB/CIDR 2026 work on BIRD and Spider 2.0-Snow found pervasive annotation errors and showed leaderboard rankings can move substantially after corrections.

Reference:

- Jin et al., *Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards*, 2026. https://arxiv.org/abs/2601.08778

This is highly relevant philosophically, but must not be conflated with database inconsistency.

We need to distinguish at least three error sources:

```text
QUERY FAULT
    generated SQL misunderstands the user intent

DATA FAULT
    database instance violates the integrity/business assumptions

BENCHMARK / LABEL FAULT
    evaluation question, gold SQL, or gold answer is wrong/ambiguous
```

The first `Fault-or-Data?` paper should control the third source rather than attempt all three simultaneously.

---

# 3. Why the problem matters specifically for SQL agents

A conventional self-correcting SQL agent observes something suspicious:

```text
empty result
unexpected duplicate rows
implausibly large aggregate
join explosion
inconsistent output samples
```

and then changes the SQL.

That policy implicitly assumes:

```text
unexpected result => query is wrong
```

But this implication is false.

Example:

```text
Intent:
    one row per active subscription

Generated SQL:
    correct

Database:
    two rows both marked is_current = true
    for the same subscription_id
```

A repair agent may respond by adding `DISTINCT`, arbitrarily selecting one row, or changing the join. The modified query can hide a source-data anomaly and become semantically wrong.

This motivates a new reliability objective:

> **Do not minimize query error alone. Minimize incorrect intervention.**

The most dangerous failure is not only a wrong answer; it is a system that confidently “fixes” the wrong layer.

---

# 4. Core benchmark: a factorial Fault-or-Data design

The cleanest evaluation construction is a controlled 2 × 2 design.

Start from a clean latent task:

```text
D*   = constraint-satisfying database
q    = natural-language question
s*   = semantically correct SQL
```

Construct four quadrants:

| Database | SQL candidate | Gold diagnostic label |
|---|---|---|
| clean | correct | OK |
| clean | faulty | QUERY_FAULT |
| inconsistent | correct | DATA_FAULT |
| inconsistent | faulty | BOTH |

A fifth label is reserved for intentionally underdetermined cases:

```text
INSUFFICIENT_EVIDENCE
```

This factorial construction has a major methodological advantage: the true source of the fault is known by construction.

It lets us measure whether the agent diagnoses the *cause* rather than merely observing that the final answer differs.

---

# 5. Data-fault injection taxonomy

The first version should emphasize **constraint-grounded faults**, not arbitrary noise.

## 5.1 Tier A — easiest, high-control inconsistencies

### Primary-key conflicts

```text
customer_id = 42, status = active
customer_id = 42, status = inactive
```

Same key, contradictory facts.

### Unique-key conflicts

Two entities share an identifier declared unique.

### Functional-dependency violations

Example:

```text
order_id -> customer_id
```

is violated by two tuples with the same `order_id` but different customers.

### Orphan foreign keys

A fact references a missing dimension/entity.

These faults align well with classical repair semantics and are the right starting point.

## 5.2 Tier B — enterprise analytical pathologies

### Multiple-current-row inconsistency

```text
entity_id = 7
is_current = true  -- row A
is_current = true  -- row B
```

### Overlapping validity intervals

Two supposedly non-overlapping SCD / history records overlap in time.

### Duplicate business event

A transaction/event is duplicated and causes SUM/COUNT inflation.

### Conflicting categorical state

The same entity-time pair receives mutually exclusive states.

These can often be expressed by denial constraints.

## 5.3 Tier C — postpone until later

- arbitrary numeric corruption
- free-text semantic contradictions
- missing-not-at-random data
- cross-system reconciliation errors
- unit/currency mistakes without explicit metadata

These require stronger assumptions about the “correct” repair and would blur the initial paper.

---

# 6. Query-fault mutation taxonomy

To obtain exact query-fault ground truth, mutate known-correct SQL with one controlled semantic error at a time.

Recommended operators:

1. wrong join key;
2. missing join predicate / fan-out;
3. `COUNT` vs `COUNT(DISTINCT ...)`;
4. missing status filter;
5. wrong temporal boundary (`<` vs `<=`);
6. wrong aggregation grain;
7. current-state table instead of history table;
8. wrong measure column (`gross` vs `net`);
9. inner join vs left join;
10. missing deduplication before aggregation.

Each mutation should be accepted only if it changes task semantics on at least one clean witness database.

This avoids “faults” that are syntactically different but semantically equivalent.

---

# 7. The `CertaintySQL` evidence engine

A practical first architecture can be modest.

```text
Question + Schema + Candidate SQL
               |
               v
      Semantic obligation extractor
               |
               v
       Constraint / provenance slice
               |
        +------+------+ 
        |             |
        v             v
 Query verifier   Data consistency analyzer
        |             |
        |             +--> violated constraints
        |             +--> conflict witnesses
        |             +--> CQA / range-CQA result
        |             +--> optional operational support probability
        |
        +-------------+
               |
               v
      Fault-or-Data classifier
               |
     +---------+----------+----------+
     |                    |          |
 QUERY_FAULT          DATA_FAULT    BOTH
     |                    |          |
 repair SQL        certify/report   repair + report
```

The LLM does not need to implement CQA itself. It consumes structured evidence from deterministic database modules.

---

# 8. Which CQA fragment is feasible for V1?

The first prototype should be intentionally narrow.

## Recommended V1 scope

Database:

```text
SQLite / DuckDB
```

Constraints:

```text
primary keys
unique keys
simple functional dependencies
selected denial constraints
```

Queries:

```text
select-project-join
single GROUP BY
COUNT / SUM / MIN / MAX
limited self-joins
```

Repair semantics:

```text
subset / tuple-deletion repairs
```

Avoid initially:

```text
attribute-value repair
arbitrary TGDs
full SQL with recursive CTEs
window-heavy queries
procedural SQL
CRUD
```

### Why this scope is justified

For primary-key settings, substantial tractable query classes are known; LinCQA shows that useful acyclic SPJ classes can be rewritten efficiently. For harder cases, CAvSAT demonstrates a SAT-based fallback. Aggregate CQA systems and 2024–2026 range-CQA results provide a principled route for numerical queries.

Therefore a prototype does **not** require solving general CQA.

---

# 9. Repair semantics are a scientific variable, not an implementation detail

“Repair” is not unique.

Possible semantics include:

- subset repairs: inclusion-maximal consistent subsets;
- cardinality repairs: retain the maximum number of facts;
- weighted / soft repairs;
- attribute-based repairs;
- operational repairs.

For the first benchmark, use **subset repairs** as the primary semantics because they are classical and easy to explain.

Then include a small sensitivity analysis:

```text
subset repair
vs
cardinality repair
vs
operational repair probability
```

A result that changes dramatically under repair semantics is itself informative:

```text
status = REPAIR_SEMANTICS_SENSITIVE
```

Long-term, this may become another agent action:

> “The data is inconsistent, and the answer depends on how the organization chooses to resolve the conflict.”

But that is beyond V1.

---

# 10. Integrity constraints: known vs discovered

CQA requires constraints. This is a major practical question.

## 10.1 V1 — oracle / declared constraints

Use constraints explicitly provided by the benchmark generator:

- PK
- UNIQUE
- FK where relevant
- small set of domain denial constraints

This cleanly isolates the research question:

> Given valid integrity assumptions, can the SQL agent distinguish query faults from data faults?

## 10.2 V2 — discovered constraints

Constraint discovery is itself an active database research area.

Relevant work:

- classical FD / denial-constraint discovery;
- approximate denial constraints for noisy databases;
- `Guardrail: Automated Integrity Constraint Synthesis From Noisy Data`, PACMMOD 2025;
- HoloClean-style cleaning systems that combine denial constraints with statistical evidence.

References:

- Chu, Ilyas & Papotti, *Discovering Denial Constraints*.
- Livshits et al., *Approximate Denial Constraints*, 2020. https://arxiv.org/abs/2005.08540
- Ma et al., *Guardrail: Automated Integrity Constraint Synthesis From Noisy Data*, PACMMOD 2025.
- HoloClean: https://www.holoclean.io/

A later paper can study the end-to-end problem:

```text
noisy DB
 -> discover likely constraints
 -> identify violations
 -> quantify answer certainty
 -> diagnose query vs data fault
```

But combining constraint discovery with agent diagnosis in the first paper would create too many confounds.

---

# 11. Strongest baseline suite

A paper in 2026 cannot compare only against one-shot LLM diagnosis.

## B0 — No diagnosis

Always trust the query result.

## B1 — Repair-first SQL agent

If execution/result looks suspicious, ask an LLM to revise the SQL.

This baseline is essential because the main claim is that repair-first behavior can create **misrepairs**.

## B2 — Direct LLM diagnosis

Provide:

- question
- schema
- candidate SQL
- sampled rows / result statistics
- declared constraints

Ask for the 4-way label directly.

## B3 — Violation-count heuristic

If any constraint violation touches referenced tables, predict DATA_FAULT; otherwise QUERY_FAULT.

This tests whether formal CQA adds value beyond simply detecting dirty tables.

## B4 — Data-quality detector + LLM

Provide exact conflict witnesses but no cross-repair answer analysis.

## B5 — CQA-only

If candidate answer changes across repairs, predict data involvement.

This is deliberately limited: a wrong query can still be stable across repairs.

## Proposed — Hybrid Fault-or-Data

Combine:

- semantic query verifier;
- relevant-constraint violation witnesses;
- CQA / range-CQA;
- final calibrated diagnostic controller.

---

# 12. Primary metrics

The first paper should pre-register a small set of primary endpoints.

## Primary 1 — 4-way diagnostic macro-F1

```text
QUERY_FAULT
DATA_FAULT
BOTH
OK
```

`INSUFFICIENT_EVIDENCE` can be introduced in a second experiment or selective setting.

## Primary 2 — Misrepair rate

On the `DATA_FAULT + correct SQL` quadrant:

```text
misrepair_rate =
    fraction of episodes where the agent changes a correct SQL
    into a semantically incorrect or less faithful SQL
```

This is probably the most novel and interpretable metric.

## Primary 3 — Correct intervention rate

```text
QUERY_FAULT -> repair SQL
DATA_FAULT  -> preserve SQL + report/quantify uncertainty
BOTH        -> repair SQL + report data issue
OK          -> answer
```

## Secondary metrics

- task success after intervention;
- accepted-answer error rate;
- CQA coverage;
- width of aggregate uncertainty interval;
- number of inspected rows;
- DB calls;
- LLM tokens;
- latency;
- diagnosis calibration.

---

# 13. Killer experiments

## Experiment A — Does self-repair damage correct SQL on dirty data?

Construct 500–1,000 `DATA_FAULT-only` episodes.

Run strong repair agents with deliberately suspicious symptoms:

- duplicate result rows;
- unexpectedly high aggregate;
- conflicting status;
- empty result caused by orphan fact.

Measure how often the agent changes correct SQL.

If the misrepair rate is substantial, the problem is empirically established before introducing our method.

This should be Figure 1 or Table 1 of the paper.

## Experiment B — Factorial diagnosis

Balanced 2 × 2 design:

```text
clean/correct
clean/query-fault
inconsistent/correct
inconsistent/query-fault
```

Compare all baselines and the hybrid method.

This is the primary causal experiment.

## Experiment C — Aggregate certainty

Tasks involving `COUNT` and non-negative `SUM`.

Compare:

```text
single observed value
vs
CQA interval [glb, lub]
```

Measure whether reporting uncertainty prevents confident wrong business answers.

## Experiment D — Violation relevance

Inject unrelated dirty tuples/tables.

A naïve detector may scream “data fault” whenever any constraint is violated.

The proposed method should determine whether the inconsistency can actually influence the current query result.

This experiment is essential because it separates **query-aware inconsistency reasoning** from generic data-quality monitoring.

---

# 14. Benchmark construction from existing Text2SQL data

## 14.1 Local controlled benchmark first

Start from SQLite databases where we can clone and modify instances cheaply.

Potential sources:

- Spider 1.0-style databases;
- BIRD SQLite databases;
- Spider 2.0-Lite SQLite subset;
- synthetic schemas with explicit integrity constraints.

Generate a clean latent snapshot if necessary, then inject controlled inconsistencies.

## 14.2 BIRD is attractive but must be handled carefully

BIRD was explicitly designed to contain real-world, large, frequently dirty database values. It contains 95 databases across 37 domains and emphasizes value-level reasoning.

However:

> “dirty values” are not automatically the same thing as violations under a known repair semantics.

Therefore we should not simply label existing BIRD irregularities as DATA_FAULT.

Instead:

1. use BIRD schemas/questions as realistic workload seeds;
2. create controlled constraint-consistent base copies for selected tasks;
3. inject known constraint violations;
4. preserve a provenance record for every injected fault.

## 14.3 Spider 2.0

Spider 2.0 is valuable for external validity because schemas are enterprise-scale and the benchmark includes Snowflake, BigQuery, and SQLite.

For V1, use the local SQLite subset or DBT/DuckDB-style local artifacts where modifications are reproducible.

Do **not** make the first benchmark depend on mutating hosted Snowflake / BigQuery databases.

---

# 15. A benchmark episode schema

```json
{
  "task_id": "fod_000123",
  "question": "How many active subscriptions were current on 2026-06-30?",
  "clean_db_id": "subscriptions_v1_clean",
  "observed_db_id": "subscriptions_v1_pk_conflict_03",
  "candidate_sql_id": "gold_sql",
  "query_fault": null,
  "data_fault": {
    "type": "multiple_current_rows",
    "constraint": "unique(subscription_id) WHERE is_current = 1",
    "witness_ids": [4182, 4183]
  },
  "gold_label": "DATA_FAULT",
  "ordinary_answer": 10431,
  "certainty": {
    "kind": "range",
    "lower": 10430,
    "upper": 10431
  },
  "recommended_intervention": "REPORT_DATA_UNCERTAINTY"
}
```

This provides substantially richer supervision than ordinary question/SQL pairs.

---

# 16. Hypotheses and pre-registration

## H1 — Misrepair exists

**Claim:** strong execution-guided / self-correcting SQL agents modify a non-trivial fraction of semantically correct SQL queries when suspicious output is caused solely by data inconsistency.

### Falsification

Reject the motivating claim if a representative strong repair baseline has <5% misrepair rate across diverse DATA_FAULT-only episodes.

## H2 — Formal data evidence improves diagnosis

**Claim:** CQA-aware evidence improves 4-way fault-source macro-F1 over direct LLM diagnosis and violation-count baselines.

### Falsification

Reject if the improvement over the strongest non-CQA baseline is <3 absolute macro-F1 points and not statistically reliable.

## H3 — Query-aware CQA beats generic dirty-data detection

**Claim:** the system distinguishes relevant inconsistencies from unrelated constraint violations better than a data-quality detector.

### Falsification

Reject if the irrelevant-inconsistency false-positive rate is not materially lower.

## H4 — Certainty intervals improve answer safety

**Claim:** on aggregate questions, range-consistent answers reduce confident incorrect numerical responses under data inconsistency.

### Falsification

Reject if intervals are too wide to be operationally useful on most tasks, or if computation cost overwhelms the target workload.

---

# 17. Statistical methodology

Use paired episodes because every injected dirty database is derived from the same clean task.

Recommended analysis:

- bootstrap confidence intervals for macro-F1 / intervention rate;
- McNemar test for paired diagnostic correctness;
- paired bootstrap for misrepair reduction;
- report per-fault-type breakdowns;
- report CQA tractable/fallback coverage separately.

Always publish raw cost components:

```text
LLM calls
prompt/completion tokens
DB calls
SAT solver time
rewriting time
wall clock
```

A reliability improvement purchased by 100× computation is a different result from a nearly free guardrail.

---

# 18. Feasibility architecture for AutoResearchClaw

This can be prototyped without building a general-purpose CQA solver.

## Phase F0 — benchmark generator

Implement:

- clean DB cloning;
- constraint-aware fault injectors;
- SQL mutation operators;
- exact quadrant labels;
- episode manifests.

## Phase F1 — simple certainty engine

For PK/unique conflicts:

- enumerate repairs for tiny controlled instances;
- use this as a correctness oracle for tests.

This is sufficient to validate benchmark semantics.

## Phase F2 — scalable engines

Add adapters:

```text
LinCQA-style rewriting for supported SPJ fragment
SAT/MaxSAT fallback for harder denial-constraint cases
range-CQA for selected aggregates
```

## Phase F3 — diagnostic agent

Agent consumes structured evidence and emits:

```json
{
  "diagnosis": "DATA_FAULT",
  "confidence": 0.91,
  "query_action": "KEEP_SQL",
  "data_action": "REPORT_UNCERTAINTY",
  "evidence": ["pk_conflict:orders.order_id=1841"],
  "answer_status": "RANGE_CERTAIN"
}
```

## Phase F4 — AutoResearchClaw experiment loop

AutoResearchClaw can sweep:

- data-fault type;
- query-fault type;
- inconsistency rate;
- CQA backend;
- model family;
- evidence format;
- diagnostic threshold.

The output should be a Pareto analysis of reliability vs cost.

---

# 19. What should NOT be built yet

To preserve scientific clarity, postpone:

- automatic constraint discovery as a required component;
- text-derived business constraints;
- attribute-value repair;
- arbitrary warehouse SQL;
- automatic physical data cleaning;
- write/CRUD tasks;
- multi-agent committees;
- persistent organizational memory.

These are follow-on papers.

The first result should answer one question convincingly:

> **Can an agent tell when to repair the SQL and when to distrust the data?**

---

# 20. Venue framing

## Database-systems framing — strongest fit if system depth is high

A SIGMOD/VLDB-style paper would emphasize:

- formal diagnostic semantics;
- scalable CQA integration;
- query-relevance analysis;
- benchmark construction;
- solver/rewriting performance;
- aggregate certainty;
- real database workloads.

This is the most natural framing if we implement a real CQA execution layer rather than only prompt an LLM with conflict summaries.

## NLP / Text2SQL framing

An ACL/EMNLP-style paper would emphasize:

- failure analysis of self-correcting Text2SQL agents;
- new `Fault-or-Data?` benchmark;
- misrepair phenomenon;
- LLM diagnostic behavior;
- structured database evidence as tool feedback.

This path has lower systems requirements but must show broad model/agent evaluation.

## Recommendation

The research problem is strongest when presented as a **database-agent reliability paper**, sitting between Text2SQL and CQA rather than pretending to be a new CQA theory contribution.

A practical sequence is:

```text
Paper A:
Fault-or-Data benchmark + misrepair phenomenon + hybrid diagnostic agent

Paper B:
Scalable CertaintySQL engine for agentic analytics

Paper C:
Discovered/business constraints + TMS-SQL integration
```

---

# 21. Reviewer objections to prepare for now

## Objection 1 — “The inconsistencies are synthetic.”

Response requirement:

- use fault patterns grounded in real data-quality literature;
- include a smaller manually audited real-dirty-data evaluation;
- show injection distributions calibrated to observed conflict frequencies where possible.

## Objection 2 — “CQA assumes correct integrity constraints.”

Correct. The first paper should explicitly state this assumption and isolate it experimentally.

Then add a secondary noisy-constraint experiment rather than hiding the dependency.

## Objection 3 — “This is just data cleaning.”

No. Data cleaning chooses/modifies a database. The primary task here is **diagnostic and answer-time**:

- preserve the observed database;
- determine whether the query result is robust to admissible repairs;
- decide whether query repair is appropriate.

## Objection 4 — “A simple constraint-violation detector is enough.”

This is why Experiment D is mandatory. A violation in an unrelated table or row should not invalidate the current answer.

## Objection 5 — “General CQA is too expensive.”

Do not claim generality. Report supported fragments, rewriting coverage, SAT fallback coverage, and cost.

## Objection 6 — “Repair semantics are subjective.”

Treat the semantics as an explicit experimental variable; flag semantics-sensitive answers rather than concealing the choice.

---

# 22. Go / no-go decision after this audit

## GO, with a narrowed thesis.

The idea remains compelling because three literatures currently touch but do not fully solve the combined problem:

```text
Text2SQL self-correction
      +
Database inconsistency / CQA
      +
Agent diagnosis / intervention selection
```

The key novelty is **layer attribution** and **misrepair prevention**, not database repair itself.

### Highest-value first experiment

Before implementing a sophisticated CQA engine, run a small controlled pilot:

```text
100 clean tasks
x
4 data-fault types
=
400 DATA_FAULT-only episodes
```

Give strong SQL repair agents the correct SQL plus suspicious results and measure:

```text
How often do they unnecessarily change it?
```

If the rate is near zero, stop or deprioritize CertaintySQL.

If the rate is substantial, the paper has a strong empirical motivation and the next engineering step is justified.

---

# 23. Reference map

### CQA foundations / systems

- Bertossi. *Database Repairing and Consistent Query Answering*. 2011.
- Dixit & Kolaitis. *A SAT-based System for Consistent Query Answering*. 2019. https://arxiv.org/abs/1905.02828
- Fan et al. *LinCQA: Faster Consistent Query Answering with Linear Time Guarantees*. 2022. https://arxiv.org/abs/2208.12339
- Marconi & Rosati. *Consistent Query Answering for Expressive Constraints under Tuple-Deletion Semantics*. 2022. https://arxiv.org/abs/2207.09198

### Repair semantics / approximation

- Lopatenko & Bertossi. *Complexity of Consistent Query Answering under Cardinality-Based and Incremental Repair Semantics*. https://arxiv.org/abs/1605.07159
- Calautti et al. *Uniform Operational Consistent Query Answering*. https://arxiv.org/abs/2204.10592
- Calautti et al. *Combined Approximations for Uniform Operational Consistent Query Answering*. https://arxiv.org/abs/2508.15814

### Aggregation

- Dixit & Kolaitis. *Consistent Answers of Aggregation Queries using SAT Solvers*. https://arxiv.org/abs/2103.03314
- Amezian El Khalfioui & Wijsen. *Computing Range Consistent Answers to Aggregation Queries via Rewriting*. https://arxiv.org/abs/2409.01648
- Amezian El Khalfioui & Wijsen. *Computing Consistent Least Upper Bounds in Aggregate Logic*. ICDT 2026.
- *A Chase-based Approach to Consistent Answers of Analytic Queries in Star Schemas*. 2026.

### Constraint / data quality

- Chu, Ilyas & Papotti. *Discovering Denial Constraints*.
- Livshits et al. *Approximate Denial Constraints*. https://arxiv.org/abs/2005.08540
- Ma et al. *Guardrail: Automated Integrity Constraint Synthesis From Noisy Data*. PACMMOD 2025.
- HoloClean. https://www.holoclean.io/

### Text2SQL evaluation context

- Li et al. *BIRD: Can LLM Already Serve as A Database Interface?* https://arxiv.org/abs/2305.03111
- Lei et al. *Spider 2.0*. https://arxiv.org/abs/2411.07763
- Wretblad et al. *Understanding the Effects of Noise in Text-to-SQL*. ACL 2024. https://arxiv.org/abs/2402.12243
- Jin et al. *Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards*. https://arxiv.org/abs/2601.08778
- Habibollah & Rafiei. *SynSQL*. https://arxiv.org/abs/2604.27261
