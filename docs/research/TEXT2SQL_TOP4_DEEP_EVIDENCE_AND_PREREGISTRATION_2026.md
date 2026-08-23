# Text2SQL Top-4 Deep Evidence Audit and Pre-Registration Sketches (2026)

> Date: 2026-08-23  
> Status: literature-driven research spike; no runtime implementation changes.  
> Purpose: move from broad ideation to paper-candidate triage by stress-testing four directions against close prior work, identifying what is already occupied, and writing falsifiable pre-registration sketches before engineering begins.

---

## 1. Protocol

This pass deliberately uses a stricter standard than the previous idea atlases.

For each candidate we require:

1. a concrete failure mode that exists in current Text2SQL / database-agent systems;
2. a direct audit of the closest Text2SQL, SQL-debugging, database-theory, and adjacent-field literature;
3. an explicit statement of what part of the proposed idea is **not novel**;
4. a surviving contribution that changes representation, objective, evidence, or evaluation—not merely agent orchestration;
5. a pre-registered primary claim and primary endpoint;
6. a falsification condition that would make us reject the idea;
7. equal-budget baselines and ablations;
8. a reviewer-objection section written before implementation.

The four candidates entering this audit are:

- `SpectrumSQL`: test-spectrum semantic fault localization before repair;
- `CertaintySQL`: Text2SQL under inconsistent databases and query-vs-data fault diagnosis;
- `TMS-SQL`: truth-maintained organizational semantics under change;
- `ConstraintAcquisitionSQL`: active acquisition of reusable business rules across a task stream.

A major goal of this pass is to be willing to **downgrade or reject** an attractive idea when older or newer literature already occupies the contribution.

---

# 2. Executive result

The ranking changed materially after deeper search.

| Rank | Candidate | Novelty after audit | Experimental clarity | Engineering risk | Reviewer risk | Recommendation |
|---|---|---:|---:|---:|---:|---|
| 1 | CertaintySQL | 5/5 | 5/5 | 3/5 | 3/5 | flagship candidate |
| 2 | TMS-SQL | 4.5/5 | 4/5 | 3/5 | 3/5 | flagship candidate |
| 3 | ConstraintAcquisitionSQL | 4.5/5 | 4/5 | 4/5 | 3.5/5 | strong long-horizon candidate |
| 4 | SpectrumSQL | 2.5/5 | 5/5 | 2/5 | 5/5 | downgrade; use as module unless sharply reframed |

The most important negative finding is that **spectrum-based fault localization has already been applied directly to SQL**. Guo et al. published clause-level spectrum/exoneration-based localization for SQL predicates in ICST 2017 and JSS 2019, and Guo's 2018 PhD dissertation is explicitly titled *Towards Automatically Localizing and Repairing SQL Faults*. Therefore, a paper whose central claim is simply "apply SFL/Ochiai to SQL" is not novel.

The surviving SpectrumSQL opportunity is much narrower: **NL-intent-conditioned semantic fault localization for generated Text2SQL plans**, where the system must synthesize semantic probes from natural-language intent, map each probe to semantic-plan obligations, and measure whether localization prevents correct-component corruption during LLM repair. Even this is crowded by recent test-driven and fine-grained Text2SQL correction work.

By contrast, this audit did not find an obvious recent Text2SQL system whose central task is **formal consistent query answering over an inconsistent database instance** or **routing between query fault and data fault before deciding to repair SQL**. Classic CQA theory is mature; that maturity is an advantage if the paper contribution is the new Text2SQL problem/evaluation interface rather than rebranding CQA.

---

# 3. Candidate A — SpectrumSQL

## 3.1 Intended thesis

A Text2SQL repair agent should not regenerate an entire query after a failed semantic check. It should first identify which semantic-plan region is responsible for the failure, then edit only that region.

The initial proposal was to borrow spectrum-based fault localization (SBFL): verifier tests form rows, SQL/plan components form columns, test outcomes and component coverage induce suspiciousness scores, and repair is restricted to the most suspicious components.

## 3.2 Deep novelty correction: SQL SFL already exists

This is the most consequential result of the audit.

### Direct prior SQL fault-localization literature

**Guo, Motro, Li, Offutt — Localizing Faults in SQL Predicates, ICST 2017**  
DOI: https://doi.org/10.1109/ICST.2017.8

This work directly applies fault-localization ideas to SQL predicates.

**Guo, Li, Offutt, Motro — Exoneration-based fault localization for SQL predicates, Journal of Systems and Software 147 (2019), 230–245**  
https://doi.org/10.1016/j.jss.2018.10.037

The paper explicitly compares modified spectrum-based fault-localization techniques for SQL clauses and introduces exoneration-based localization using row-based dynamic slicing and delta-debugging-inspired reasoning. It evaluates 450 subject queries from five databases and includes real faults from an industrial application.

**Yun Guo — Towards Automatically Localizing and Repairing SQL Faults, PhD dissertation, George Mason University, 2018**  
https://cs.gmu.edu/~offutt/documents/theses/YunGuo-Dissertation.pdf

This title alone makes clear that generic SQL fault localization/repair is not a new research problem.

### Consequence

The following framing should be rejected:

> "We introduce spectrum-based fault localization for SQL and use suspiciousness scores to guide repair."

That would be substantially anticipated by the 2017–2019 SQL-testing literature.

## 3.3 Recent Text2SQL work makes the space even tighter

**SHARE — ACL 2025**  
https://aclanthology.org/2025.acl-long.552/

SHARE targets precise error localization and correction by converting SQL into a stepwise action trajectory and performing hierarchical action correction.

**ErrorLLM — 2026**  
https://arxiv.org/abs/2603.03742

ErrorLLM explicitly models categories of SQL errors and emphasizes that correction quality depends strongly on error identification; importantly, it targets the failure mode where a correct or mostly-correct SQL query is corrupted by unnecessary refinement.

**TS-SQL — Findings of EMNLP 2025**

TS-SQL is especially close to any "test-driven Text2SQL repair" claim. It generates test data and executable Python test code, executes SQL and test code, and uses result mismatch as feedback for revision. Its existence means that "generate semantic tests, execute them, and revise SQL" is also not sufficient novelty by itself.

**BIRD-CRITIC / SWE-SQL — NeurIPS 2025**  
https://bird-critic.github.io/  
https://github.com/bird-bench/BIRD-CRITIC-1

BIRD-CRITIC establishes SQL issue diagnosis/debugging as a benchmark problem with executable test cases, multiple dialects, CRUD tasks, and query-plan evaluation.

**Developing and Benchmarking Verification Algorithms to Improve Text-to-SQL Generation — VLDB 2026 program**  
https://vldb.org/2026/program.html

The work formalizes Text2SQL verification as a standalone problem and includes synthetic-execution-consistency verification, further raising the novelty bar for test-based verification.

## 3.4 Adjacent software-engineering evidence

SBFL remains scientifically useful as an experimental mechanism even if it is not a novel SQL mechanism.

Relevant literature includes:

- standard SBFL families such as Ochiai, Tarantula, Jaccard, DStar, and statistical debugging;
- FuseFL (2024), which combines LLM reasoning with spectrum-based results and test outcomes;
- LLM4FL (2024), which combines SBFL rankings with agentic divide-and-conquer and self-reflection;
- ProFL, which shows that repair outcomes can themselves improve fault localization.

A useful lesson from this literature is that **localization and repair can form a feedback loop**, rather than localization being a one-shot pre-processing stage.

## 3.5 Surviving gap

The only version worth carrying forward is:

### `NL-SpectrumSQL`: intent-conditioned semantic localization

The difference from old SQL SFL must be explicit:

- old SQL SFL assumes a faulty SQL program plus test executions and localizes faulty SQL predicates/clauses;
- NL-SpectrumSQL starts from a **natural-language intent contract** and an LLM-generated SQL/semantic plan;
- semantic probes must be synthesized automatically from the intent, schema, data constraints, and business rules;
- coverage is defined over **semantic obligations** rather than only runtime predicate execution;
- the goal is not only Top-k fault localization but preventing **correct-component corruption** during generative repair.

Example semantic obligations:

```text
O1 result grain = one row per customer
O2 revenue excludes refunded value
O3 date window = closed-open fiscal quarter
O4 cancelled orders excluded
O5 ranking occurs after aggregation
```

Example plan nodes:

```text
N1 entity/table selection
N2 customer-order join
N3 refund aggregation
N4 temporal predicate
N5 status predicate
N6 customer grouping
N7 net-revenue expression
N8 rank/limit
```

A verifier probe has both an outcome and an obligation-to-node incidence vector.

## 3.6 Pre-registration sketch

### Primary claim

> NL-intent-conditioned fault localization before LLM repair reduces collateral edits to correct semantic components and increases final repair success compared with whole-query repair under equal inference budget.

### Dataset construction

Start from verified-correct SQL and inject exactly one known semantic fault.

Fault operators:

1. join-key swap;
2. missing pre-aggregation on a one-to-many edge;
3. `COUNT` ↔ `COUNT DISTINCT`;
4. boundary shift (`<` ↔ `<=`);
5. wrong temporal anchor;
6. dropped status filter;
7. wrong aggregation grain;
8. wrong source table for current-vs-history state;
9. ranking before aggregation;
10. incorrect null semantics.

Each corrupted query has exact ground-truth fault location.

### Baselines

- whole-query LLM repair;
- error-message-only repair;
- AST decomposition + LLM classifier;
- SHARE-style hierarchical correction baseline where feasible;
- test-driven revision without localization;
- classic clause-level SBFL where applicable;
- NL-SpectrumSQL.

### Primary endpoint

`final_repair_success @ equal total model tokens`

### Secondary endpoints

- Top-1 / Top-3 localization accuracy;
- mean reciprocal rank of faulty node;
- fraction of originally-correct nodes modified;
- regression rate among initially correct components;
- number of LLM calls;
- verifier calls;
- latency.

### Falsification condition

Reject the central hypothesis if localization does not improve repair success or does not materially reduce correct-component corruption under equal cost.

### Go/no-go threshold

Do not make SpectrumSQL a flagship paper unless it shows both:

- >= 10 percentage-point relative reduction in correct-component corruption; and
- statistically significant repair-success gain over test-driven whole-query revision.

## 3.7 Reviewer objection

> "Spectrum-based fault localization for SQL existed in 2017–2019, and test-driven Text2SQL correction already exists."

This objection is valid. The paper survives only if it demonstrates a new **NL semantic-obligation formulation**, a new benchmark with precise semantic-fault ground truth, and a measurable downstream property that older SQL SFL did not study: preservation of correct LLM-generated semantics during repair.

### Verdict

**Downgrade from standalone flagship to enabling module / smaller paper unless experiments reveal a surprisingly strong effect.**

---

# 4. Candidate B — CertaintySQL

## 4.1 Problem statement

Current Text2SQL evaluation usually conflates two assumptions:

1. the generated query is correct;
2. the database instance is a reliable realization of the intended integrity constraints.

In production, assumption (2) often fails.

Examples:

- supposedly unique entity IDs are duplicated;
- a functional dependency is violated;
- two records are simultaneously marked `current`;
- an orphan foreign key exists;
- status-history intervals overlap;
- dimension mappings conflict;
- late-arriving facts create inconsistent snapshots.

A repair-oriented SQL agent that sees a surprising answer may therefore make the wrong diagnosis: it edits a correct query to compensate for bad data.

## 4.2 Database-theory foundation

This is not a new database-theory problem; that is precisely why it is attractive as a principled transfer.

### Consistent Query Answering (CQA)

Arenas, Bertossi, Chomicki and subsequent work define **repairs** of an inconsistent database and **consistent answers** as answers that survive across admissible repairs.

A modern systems anchor is:

**Dixit & Kolaitis — A SAT-based System for Consistent Query Answering**  
https://arxiv.org/abs/1905.02828

CAvSAT reduces CQA problems to SAT / Weighted MaxSAT and supports unions of conjunctive queries under denial constraints, including functional dependencies.

**CAvSAT: Answering Aggregation Queries over Inconsistent Databases via SAT Solving — SIGMOD 2021**  
https://research.ibm.com/publications/cavsat-answering-aggregation-queries-over-inconsistent-databases-via-sat-solving

This extends practical CQA to important aggregation settings including SUM and COUNT under key/FD/denial constraints.

For numerical aggregation, the natural answer is often a **range** across repairs rather than a single number.

**Amezian El Khalfioui & Wijsen — Computing Range Consistent Answers to Aggregation Queries via Rewriting, PACM 2024**  
https://arxiv.org/abs/2409.01648

**Computing Consistent Least Upper Bounds in Aggregate Logic — ICDT 2026**  
https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ICDT.2026.4

This literature provides both semantics and computational machinery for exposing data-induced uncertainty.

## 4.3 Closest modern SQL-agent work

**BIRD-CRITIC / SWE-SQL** evaluates whether LLMs can diagnose and solve realistic SQL issues. It includes test cases, multiple dialects, execution plans, and agentic debugging. However, the central evaluation target is solving the SQL/database user issue, not formal answer semantics across multiple repairs of an inconsistent database.

**Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards — 2026**  
https://arxiv.org/abs/2601.08778

This work shows that the *evaluation artifacts* themselves can be inconsistent/wrong: expert analysis reports high annotation error rates in benchmark subsets and demonstrates substantial leaderboard effects. It strengthens the broader motivation that "database/benchmark evidence is always trustworthy" is unsafe, although annotation error is distinct from runtime database inconsistency.

Recent verification work focuses on whether SQL matches NL intent. That is orthogonal to the question here: **what should the answer mean when the database violates assumptions required by the intent?**

## 4.4 Search result / novelty assessment

A targeted search did not surface an obvious recent Text2SQL system whose main contribution is to combine LLM-generated NL-to-SQL with formal CQA semantics over inconsistent database instances and explicitly decide whether the next action should be query repair or data-uncertainty reporting.

This is not a proof of absence. The claim should therefore be phrased conservatively in any paper: "we did not identify prior Text2SQL work that studies this interface as the primary task," not "no prior work exists."

## 4.5 Proposed task: `Fault-or-Data?`

Given:

```text
natural-language request q
schema S
integrity/business constraints C
database instance D
candidate SQL P
```

predict one of:

```text
QUERY_FAULT
DATA_FAULT
BOTH
INSUFFICIENT_EVIDENCE
```

and return the appropriate action:

```text
repair SQL
report data uncertainty
repair SQL + report data uncertainty
ask / abstain
```

This diagnostic routing is more novel than simply placing CAvSAT behind an LLM.

## 4.6 Answer contract

For inconsistent-data cases, the response can contain:

```yaml
query_status: semantically_supported
data_status: inconsistent_under_constraints
certain_rows: ...
possible_rows: ...
aggregate_range:
  lower: ...
  upper: ...
violated_constraints:
  - ...
minimal_witnesses:
  - ...
action: report_uncertainty_not_query_repair
```

For a finance aggregate, returning `[9.7M, 11.2M]` with the conflicting records that create the interval may be more truthful than forcing a single number.

## 4.7 Benchmark design — `DirtyBIRD` / `CertaintyBench-SQL`

Start from a clean benchmark task with validated SQL and database.

Inject controlled inconsistencies while keeping user intent fixed:

1. primary-key duplicate;
2. functional-dependency conflict;
3. mutually incompatible current records;
4. overlapping validity intervals;
5. orphan relation;
6. conflicting categorical mappings;
7. duplicated bridge rows;
8. late-arriving contradictory event;
9. inconsistent currency/source-of-truth mapping;
10. conflicting business-rule evidence.

Each perturbation has a known integrity constraint and repair set or known answer range for the supported fragment.

Crucially, include matched **query-fault** cases and **both-fault** cases so that a model cannot solve the benchmark by always blaming data.

## 4.8 Pre-registration sketch

### Primary claim

> A query-vs-data diagnostic gate plus CQA-aware answer semantics reduces false-certainty and unnecessary SQL rewrites on inconsistent databases without materially degrading clean-database Text2SQL accuracy.

### Baselines

- ordinary Text2SQL agent;
- self-refinement / SQL repair agent;
- verifier-only agent;
- data-quality detector + ordinary SQL answer;
- oracle constraint checker without CQA;
- CQA-aware diagnostic agent.

### Primary endpoint

`false_certainty_rate` on inconsistent-data tasks.

A response is falsely certain when it presents a single definitive answer even though multiple admissible repairs imply different results or the output tuple is not certain.

### Secondary endpoints

- query-vs-data fault classification accuracy;
- unnecessary SQL edit rate on correct-query / dirty-data cases;
- certain-answer precision;
- answer coverage;
- aggregate interval coverage and width;
- clean-data accuracy degradation;
- computation cost;
- explanation/witness validity.

### Critical new metric: `misrepair rate`

```text
misrepair_rate =
  (# correct SQL queries modified because of data inconsistency)
  / (# correct SQL queries presented with data inconsistency)
```

This metric directly tests whether contemporary self-correction has the wrong action prior.

### Falsification condition

Reject the central thesis if an ordinary verifier/repair agent can achieve comparable false-certainty and misrepair rates without explicit query-vs-data modeling or CQA semantics.

### Go/no-go threshold

Proceed to full paper if the benchmark exposes a large misrepair failure mode and the proposed method reduces false certainty by >= 30% relative while losing <= 2 absolute points on clean tasks.

## 4.9 Reviewer objections and defenses

### Objection 1

> "This is just CQA plus an LLM frontend."

**Defense:** the paper contribution must be the interface problem and empirical failure mode:

- NL/schema/docs → integrity-semantic selection;
- query-fault vs data-fault routing;
- benchmark construction with matched query/data/both faults;
- misrepair analysis of modern SQL agents;
- user-facing certain/possible/range answer contract.

### Objection 2

> "Integrity constraints are rarely complete in enterprise databases."

**Defense:** make constraint provenance a first-class variable and evaluate three regimes:

- explicit DB constraints;
- explicit + documentation-derived constraints;
- uncertain candidate constraints with confidence.

### Objection 3

> "CQA is computationally expensive."

**Defense:** report a coverage frontier. Use direct SQL rewrites for tractable classes, SAT/MaxSAT for supported harder cases, and abstention / approximate bounds for unsupported cases.

### Verdict

**Highest-priority flagship candidate.** It joins mature database theory to a current LLM-agent failure mode while creating a new evaluation axis—false certainty under dirty data—rather than another generation architecture.

---

# 5. Candidate C — TMS-SQL

## 5.1 Problem statement

Current enterprise Text2SQL agents increasingly use memory, but "remember more" is not enough.

A stored belief can become invalid because:

- a business metric definition changes;
- a source-of-truth table changes;
- a fiscal calendar changes;
- an exception policy is introduced;
- a schema split/merge changes a mapping;
- an old document remains retrievable after a newer policy supersedes it.

The hard problem is not retrieval. It is **dependency-aware retraction and revision**.

Example:

```text
B1: revenue_source = invoice.net_amount
B2: orders.customer_id -> customers.id
B3: fiscal_year_starts_in_february
```

Suppose a new finance policy invalidates B1. A robust system should invalidate metrics/query fragments derived from B1 while preserving B2 and B3 unless they also depend on the changed evidence.

## 5.2 Text2SQL memory frontier

### AgentSM — 2026

**AgentSM: Semantic Memory for Agentic Text-to-SQL**  
https://arxiv.org/abs/2601.15709

AgentSM stores interpretable semantic programs derived from prior execution traces and uses them to guide future agent reasoning. It reports substantial efficiency gains and 44.8% execution accuracy on Spider 2.0 Lite.

### MIRA — August 2026

**MIRA: Evidence-Verified Repair Memory for Text-to-SQL Correction**  
https://arxiv.org/abs/2608.06950

MIRA substantially raises the novelty bar. It decomposes historical corrections into independently reusable repair-memory items, retrieves candidate items for a current SQL query, verifies their support against current database evidence, and adapts supported repairs. It reports large execution-accuracy improvements on BIRD and ScienceBenchmark.

Therefore, these claims are no longer sufficiently novel:

- "store structured Text2SQL memories";
- "store failure/repair memories";
- "validate retrieved memories against the current DB";
- "reuse historical correction fragments."

### EvoSchema — PVLDB 2025 / arXiv release 2026

**EvoSchema: Towards Text-to-SQL Robustness Against Schema Evolution**  
https://www.vldb.org/pvldb/vol18/p3655-zhang.pdf

EvoSchema systematically perturbs schemas with ten evolution types and shows that table-level changes are particularly damaging. This means a TMS-SQL paper cannot claim novelty merely from "Text2SQL under schema changes."

## 5.3 Agent-memory literature in 2026

The timing is favorable because multiple independent works show that stale-memory handling is still unsolved.

**STALE: Can LLM Agents Know When Their Memories Are No Longer Valid? — 2026**  
https://arxiv.org/abs/2605.06527

STALE studies implicit conflicts where later observations invalidate old memories without explicit negation. It reports that even strong systems struggle, and introduces CUPMem with write-side state adjudication / propagation-aware retrieval.

**Supersede: Diagnosing and Training the Memory-Update Gap in LLM Agents — 2026**  
https://arxiv.org/abs/2606.27472

Supersede isolates temporal fact supersession as a distinct memory-maintenance problem and provides a trainable environment targeting use of current rather than stale values.

**From Recall to Forgetting: Benchmarking Long-Term Memory for Personalized Agents — ACL 2026**  
https://arxiv.org/abs/2604.20006

Memora introduces Forgetting-Aware Memory Accuracy and finds that memory systems frequently reuse invalid memories.

**OAKS: Can Large Language Models Keep Up? — ACL 2026**  
https://aclanthology.org/2026.acl-long.1956/

OAKS evaluates online adaptation to continual knowledge streams and finds significant state-tracking/adaptation limitations in frontier models and agentic memory systems.

These papers make stale-state memory a current research problem. They also mean that TMS-SQL must go beyond "detect stale memory and overwrite it."

## 5.4 Classic mechanism: Truth Maintenance Systems

The conceptual foundation is Jon Doyle's truth/reason-maintenance work: maintain explicit justifications for beliefs and retract dependent conclusions when supporting assumptions no longer hold.

A TMS-like architecture distinguishes:

- base evidence;
- assumptions;
- derived beliefs;
- justifications;
- contradiction handling;
- dependency-directed invalidation.

This is more specific than a generic graph memory.

## 5.5 Surviving contribution: semantic justification graph

### Core representation

```yaml
belief_id: metric.revenue.v3
claim: revenue = captured_payment - settled_refund - chargeback
scope:
  department: finance
  region: global
validity:
  from: 2026-01-01
  to: null
evidence:
  - finance_policy_2026_v3
justifications:
  - payment_source_is_canonical
  - refunds_use_settlement_date
supersedes:
  - metric.revenue.v2
```

Derived artifact:

```yaml
artifact_id: sql_fragment.net_revenue
requires:
  - metric.revenue.v3
  - join.payment_order.current
```

If one root belief changes, invalidation follows the explicit dependency graph.

### Key distinction

TMS-SQL is not primarily a retrieval architecture. It is a **semantic artifact lifecycle system**:

```text
new evidence
   ↓
conflict / supersession detection
   ↓
minimal belief revision
   ↓
dependency propagation
   ↓
selective invalidation of cached plans / SQL / memories
   ↓
re-derive only affected artifacts
```

## 5.6 New metric: semantic blast radius

A central measurable property is how precisely a memory update propagates.

Let:

- `G*` be the ground-truth dependency graph;
- `A` be the set of artifacts that truly depend on a changed belief;
- `I` be artifacts invalidated by the system.

Measure:

```text
blast_radius_precision = |I ∩ A| / |I|
blast_radius_recall    = |I ∩ A| / |A|
```

This creates a stronger target than simple "uses newest fact."

Over-invalidation wastes memory and destroys useful knowledge; under-invalidation preserves stale semantics.

## 5.7 Benchmark: `LivingSemantics`

Construct a synthetic-but-controlled organization over a timeline.

State includes:

- 20–50 business definitions;
- 10–30 join/source-of-truth conventions;
- fiscal/time policies;
- exceptions;
- department scopes;
- schema versions;
- documents with different authority levels.

Events:

1. policy supersession;
2. exception insertion;
3. temporary override;
4. schema rename;
5. table split/merge;
6. source-of-truth migration;
7. stale document remains available;
8. conflicting lower-authority source appears;
9. policy rollback;
10. partial update affecting only one region/team.

After each event, ask a mixture of repeated and novel Text2SQL tasks.

## 5.8 Pre-registration sketch

### Primary claim

> Explicit justification graphs with dependency-directed invalidation reduce stale-semantic reuse while preserving unaffected SQL knowledge better than overwrite-, retrieval-, and consolidation-based memory baselines.

### Baselines

- no persistent memory;
- vector/RAG memory;
- latest-fact overwrite;
- structured semantic memory (AgentSM-inspired);
- evidence-checked repair memory (MIRA-inspired);
- state-consolidation baseline inspired by modern memory systems;
- TMS-SQL.

### Primary endpoint

`stale_semantic_action_rate`

Fraction of tasks whose SQL/answer uses a belief that should have been invalidated by prior evidence.

### Secondary endpoints

- blast-radius precision / recall;
- unaffected-knowledge preservation;
- memory churn;
- current-task execution correctness;
- future-task correctness;
- time-to-recovery after semantic change;
- stale-premise resistance;
- explanation fidelity: can the system name the evidence chain supporting the current SQL?

### Counterfactual test

Ask:

> "If finance policy v3 were revoked and v2 restored, which cached metrics and SQL artifacts would change?"

A justification graph should support deterministic dependency tracing; ordinary semantic retrieval should not reliably provide the exact affected set.

### Falsification condition

Reject the TMS-specific claim if a simpler timestamped overwrite or modern consolidation memory achieves statistically indistinguishable stale-semantic rate and blast-radius accuracy.

### Go/no-go threshold

Proceed if TMS-SQL improves stale-semantic action rate by >= 25% relative and retains >= 95% of unaffected valid artifacts after targeted policy changes.

## 5.9 Reviewer objections

### Objection 1

> "Truth-maintenance systems are old AI."

Correct. The novelty is not TMS itself. The paper needs to show that modern Text2SQL memories produce a measurable dependency-revision failure that TMS-style justifications solve.

### Objection 2

> "STALE/Supersede already study memory updates."

The distinction must be domain and representation specific:

- business rules rather than isolated personal facts;
- many downstream compiled query artifacts depend on one belief;
- multiple authority/scoping dimensions;
- exact semantic blast-radius evaluation;
- SQL behavior, not only QA over memories.

### Objection 3

> "EvoSchema already covers change."

EvoSchema concerns schema evolution robustness. TMS-SQL should include schema changes but center on **organizational semantic dependencies and derived artifacts**, including policy, definition, source-of-truth, and exception changes.

### Verdict

**Strong flagship candidate.** It is timely because Text2SQL memory is rapidly advancing while general agent-memory work is simultaneously exposing supersession/forgetting failures.

---

# 6. Candidate D — ConstraintAcquisitionSQL

## 6.1 Problem statement

Most interactive Text2SQL systems treat clarification as an expense paid to solve the current query.

Example:

```text
User: Show active customers.
Agent: Does active mean activity in the last 90 days?
User: Usually yes, but annual-contract customers remain active until contract expiry.
```

A conventional interactive system resolves this one task. A long-lived enterprise agent should instead infer a reusable rule:

```text
active_customer(c) :=
    activity_within_90d(c)
    OR active_annual_contract(c)
```

and use that rule on later tasks.

The research objective changes from **current-query information gain** to **task-stream semantic learning**.

## 6.2 Current interactive Text2SQL frontier

### Expected Information Gain — 2025

**Interactive Text-to-SQL via Expected Information Gain for Disambiguation**  
https://arxiv.org/abs/2507.06467

The system maintains a distribution over candidate SQL queries and asks a clarification about the branching decision expected to reduce uncertainty most. This occupies a strong formulation of per-task active clarification.

### PRACTIQ — NAACL 2025

PRACTIQ studies ambiguous/unanswerable conversational Text2SQL and multi-turn clarification.

### CLARITY — ACL Industry 2026

CLARITY studies multi-faceted ambiguities and diverse user behaviors, showing that models still struggle with precise localization/resolution of ambiguity sources.

### PleaSQLarify — CHI 2026

PleaSQLarify studies pragmatic clarification/repair using interpretable decision variables and human-facing interaction design.

### Value of Information for agents — ACL 2026

General VoI work explicitly trades expected utility gain against user/cognitive cost.

### BIRD-Interact — ICLR 2026

BIRD-Interact makes long, tool-using and conversational interaction a benchmark target and demonstrates that interaction-time scaling remains difficult.

### Consequence

The following framing is too occupied:

> "Ask the user a clarification question when uncertainty is high / when expected information gain is large."

## 6.3 Adjacent-field mechanism: interactive constraint acquisition

Constraint acquisition aims to recover an unknown constraint model by asking a user informative queries.

**Tsouros, Berden, Guns — Guided Bottom-Up Interactive Constraint Acquisition, CP 2023**  
https://arxiv.org/abs/2307.06126

The work reduces interaction and scales to much larger candidate sets; reported experiments reduce required queries by up to 60% in studied settings.

**Tsouros, Berden, Guns — Learning to Learn in Interactive Constraint Acquisition, AAAI 2024**  
https://arxiv.org/abs/2312.10795

It uses statistical ML predictions throughout query generation, scope finding, and constraint identification, reporting up to 72% reduction in queries required to converge.

**Tsouros et al. — Generalizing Constraint Models in Constraint Acquisition / GenCon, 2024–2025**  
https://arxiv.org/abs/2412.14950

GenCon addresses an especially important limitation: moving from instance-specific ground constraints to **parameterized constraint specifications** that generalize to varying instances, with interpretable learned rules and robustness to noise.

This is the mechanism that makes the transfer more than a metaphor.

## 6.4 Surviving contribution

### `OrgConSQL`: active acquisition of organization-wide semantic rules

The hidden target is not one SQL query. It is an evolving set of reusable semantic constraints:

```yaml
rule: active_customer
scope:
  region: global
definition:
  any_of:
    - last_activity_days <= 90
    - annual_contract_active == true
exceptions:
  - compliance_suspended == true
```

Another rule:

```yaml
rule: recognized_revenue
scope:
  department: finance
expression:
  captured_payment - settled_refund - chargeback
valid_from: 2026-01-01
```

Each user interaction can update this rule base.

## 6.5 New objective: future-task-aware question value

Per-task EIG asks which question most reduces uncertainty for the current SQL distribution.

OrgConSQL instead optimizes:

```text
Value(question) =
    current_task_gain
  + beta * expected_future_task_gain
  + gamma * rule_generalization_gain
  - lambda_user * interaction_cost
  - lambda_risk * wrong_rule_risk
```

A question may be suboptimal for the current query but valuable because it identifies a high-frequency organization rule.

## 6.6 Benchmark: `TeachSQL-Stream`

Generate or curate a persistent enterprise environment.

### Hidden semantic model

- 20–50 reusable business rules;
- parameterized definitions;
- 5–20 exception rules;
- scope by region/team/time;
- rule frequency distribution;
- some highly reusable rules and some one-off rules.

### Task stream

500–2,000 Text2SQL requests against the same organization.

Task types deliberately recur at the **rule** level but vary at the NL/question level.

### User oracle

The user simulator can:

- answer correctly;
- say "I don't know";
- provide noisy answer with configurable probability;
- give only a local example instead of a general rule;
- correct an earlier statement;
- charge different cognitive costs for different question types.

## 6.7 Pre-registration sketch

### Primary claim

> Under a fixed human-interaction budget, active acquisition of reusable organization-level semantic constraints achieves lower cumulative Text2SQL regret than per-task clarification policies.

### Baselines

- never ask;
- always ask;
- uncertainty threshold;
- per-task Expected Information Gain;
- general Value-of-Information clarification;
- passive memory of user corrections;
- active ground-constraint acquisition;
- active acquisition + parameterized generalization (OrgConSQL).

### Primary endpoint

`cumulative semantic regret` over the task stream.

For each task, regret measures loss relative to an oracle with the true organizational rule set, plus weighted interaction cost.

### Secondary endpoints

- task success over time;
- user questions per 100 tasks;
- recovered rule precision/recall;
- rule-scope accuracy;
- exception recall;
- transfer to unseen questions that instantiate learned rules;
- robustness to noisy/refused answers;
- rule acquisition latency;
- negative transfer from over-generalized rules.

### Killer experiment

Give all methods exactly the same total user-turn budget.

Plot cumulative task success vs task index.

A true cross-task acquisition system should start similarly to per-task EIG but eventually pull away because earlier questions reduce the need for later clarification.

The desired signature is a **learning curve over organization semantics**, not a one-query accuracy bump.

### Falsification condition

Reject the core claim if per-task EIG plus a simple correction memory matches active rule acquisition over long streams at equal interaction cost.

### Go/no-go threshold

Proceed if OrgConSQL produces:

- statistically significant lower cumulative regret;
- >= 30% fewer clarification turns in the second half of the stream at matched accuracy; and
- positive transfer to unseen NL formulations of already learned rules.

## 6.8 Reviewer objections

### Objection 1

> "The synthetic organization rules are artificial."

Mitigation:

- derive rule templates from real enterprise-style benchmark documentation;
- use BIRD-Interact knowledge structures where licensing/evaluation permits;
- draw rule motifs from enterprise business-logic datasets and public SQL corpora;
- have domain practitioners validate a subset;
- include a smaller human-authored evaluation set.

### Objection 2

> "This is active learning, not Text2SQL."

That is partly the point. The paper should show that recurring enterprise Text2SQL is better modeled as **interactive semantic model acquisition** than independent NL-to-program generation.

### Objection 3

> "What if rules change?"

This naturally connects to TMS-SQL, but the first paper should avoid scope explosion. The primary OrgConSQL paper can use a mostly stationary rule phase plus a small drift section; dynamic revision can be a separate follow-up.

### Verdict

**Strong candidate, especially for a long-horizon benchmark paper.** Its novelty depends on optimizing future task streams and acquiring parameterized reusable rules, not simply asking better clarification questions.

---

# 7. Cross-candidate synthesis

The top candidates can be viewed as attacking different assumptions of classical Text2SQL.

| Classical assumption | Candidate that attacks it |
|---|---|
| DB instance is trustworthy | CertaintySQL |
| remembered semantics stay valid | TMS-SQL |
| ambiguity is local to the current query | ConstraintAcquisitionSQL |
| repair can safely rewrite the whole query | NL-SpectrumSQL |

This suggests a broader research thesis:

> Enterprise Text2SQL fails not only because generation is hard, but because its **epistemic state** is underspecified: the agent must know which semantic rules it has learned, why they are believed, whether they still hold, whether the data satisfies them, and which component is responsible when a result is wrong.

That is more differentiated than adding another planner/critic/reviewer loop.

---

# 8. Recommended publication sequence

## Paper 1 — `When Query Repair Is the Wrong Action: Text2SQL Under Inconsistent Databases`

Core:

- `CertaintyBench-SQL` / `DirtyBIRD`;
- matched query-fault, data-fault, both-fault cases;
- misrepair and false-certainty metrics;
- CQA-aware diagnostic router;
- certain / possible / interval answers.

Why first:

- clear conceptual gap;
- mature formal foundation;
- strong database-systems identity;
- benchmark contribution stands even if the first agent is simple.

## Paper 2 — `Living Semantics: Truth-Maintained Memory for Enterprise Text2SQL`

Core:

- semantic justification graph;
- dependency-directed invalidation;
- semantic blast radius;
- `LivingSemantics` benchmark;
- comparison to modern agent-memory baselines.

Why second:

- timely 2026 memory-update literature;
- directly extends the enterprise-agent story;
- can reuse semantic contracts and provenance infrastructure from Paper 1.

## Paper 3 — `Teach Once, Query Many: Active Organizational Constraint Acquisition for Text2SQL`

Core:

- hidden reusable organization rules;
- future-task-aware interaction objective;
- parameterized rule induction;
- cumulative regret / clarification economy.

Why third:

- highest long-horizon ambition;
- needs a more sophisticated user simulator / stream benchmark;
- naturally integrates later with TMS-SQL when learned rules change.

## SpectrumSQL role

Implement only as an internal diagnostic module or a narrowly scoped workshop/system study unless early experiments demonstrate a large preservation-of-correct-semantics effect that recent Text2SQL correctors do not achieve.

---

# 9. Minimal shared experimental substrate

Before implementing any full paper method, build a small research substrate that all three flagship directions can reuse.

This is not yet an implementation request; it is the experimental interface that future design should target.

```text
SemanticContract
  - business rules
  - result grain
  - integrity assumptions
  - temporal assumptions
  - provenance / authority

DatabaseWorld
  - clean snapshot
  - controlled inconsistency operators
  - schema/version timeline

TaskStream
  - NL requests
  - rule dependencies
  - expected semantic answer

EvidenceGraph
  - source -> belief
  - belief -> plan
  - plan -> SQL fragment

Evaluation
  - execution success
  - semantic correctness
  - false certainty
  - misrepair
  - stale semantic use
  - blast radius
  - clarification cost
  - cumulative regret
```

This substrate would let AutoResearchClaw run controlled studies without prematurely committing to one giant SQL-agent architecture.

---

# 10. Stop/go rules before engineering

To prevent idea momentum from overriding evidence, use these rules:

### CertaintySQL

**GO** if we can construct at least 100 high-quality matched tasks where a correct query becomes answer-uncertain only because of controlled data inconsistency, and ordinary repair agents exhibit a measurable misrepair/false-certainty problem.

**STOP / REFRAME** if modern agents naturally detect these data faults without special modeling or if CQA coverage is too narrow to support realistic analytics.

### TMS-SQL

**GO** if existing Text2SQL/agent-memory baselines show stale downstream SQL artifacts after targeted semantic-policy changes and simple overwrite cannot preserve unaffected knowledge.

**STOP / REFRAME** if timestamp + latest-value consolidation performs as well as explicit dependencies.

### ConstraintAcquisitionSQL

**GO** if recurring business-rule structure produces substantial cross-task transfer and active questioning reduces later user interaction.

**STOP / REFRAME** if a simple passive correction memory absorbs nearly all benefit.

### SpectrumSQL

**GO as standalone** only if NL-conditioned semantic localization materially outperforms both modern Text2SQL localization/correction and classic SQL SFL on a new semantic-fault benchmark.

Otherwise, keep it as supporting infrastructure.

---

# 11. Evidence index

## SpectrumSQL / fault localization

1. Guo, Motro, Li, Offutt. *Localizing Faults in SQL Predicates*. ICST 2017. https://doi.org/10.1109/ICST.2017.8
2. Guo, Li, Offutt, Motro. *Exoneration-based fault localization for SQL predicates*. JSS 2019. https://doi.org/10.1016/j.jss.2018.10.037
3. Guo. *Towards Automatically Localizing and Repairing SQL Faults*. PhD dissertation, 2018. https://cs.gmu.edu/~offutt/documents/theses/YunGuo-Dissertation.pdf
4. Qu et al. *SHARE*. ACL 2025. https://aclanthology.org/2025.acl-long.552/
5. Hong et al. *ErrorLLM*. 2026. https://arxiv.org/abs/2603.03742
6. BIRD-CRITIC / SWE-SQL. https://bird-critic.github.io/
7. BIRD-CRITIC repository. https://github.com/bird-bench/BIRD-CRITIC-1
8. VLDB 2026 verification program entry. https://vldb.org/2026/program.html
9. Widyasari et al. *FuseFL*. 2024. https://arxiv.org/abs/2403.10507
10. Rafi et al. *LLM4FL*. 2024. https://arxiv.org/abs/2409.13642

## CertaintySQL / inconsistent databases

1. Dixit & Kolaitis. *A SAT-based System for Consistent Query Answering*. https://arxiv.org/abs/1905.02828
2. CAvSAT aggregation, SIGMOD 2021. https://research.ibm.com/publications/cavsat-answering-aggregation-queries-over-inconsistent-databases-via-sat-solving
3. Amezian El Khalfioui & Wijsen. *Computing Range Consistent Answers to Aggregation Queries via Rewriting*. https://arxiv.org/abs/2409.01648
4. *Computing Consistent Least Upper Bounds in Aggregate Logic*. ICDT 2026. https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.ICDT.2026.4
5. Amezian El Khalfioui & Wijsen. *Consistent Query Answering for Primary Keys and Conjunctive Queries with Counting*. https://arxiv.org/abs/2211.04134
6. BIRD-CRITIC / SWE-SQL. https://bird-critic.github.io/
7. Jin et al. *Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards*. https://arxiv.org/abs/2601.08778
8. Classic CQA survey material by Chomicki/Bertossi and subsequent repairs/CQA literature.

## TMS-SQL / dynamic memory

1. Biswal et al. *AgentSM*. https://arxiv.org/abs/2601.15709
2. Liu et al. *MIRA*. https://arxiv.org/abs/2608.06950
3. Zhang et al. *EvoSchema*. https://www.vldb.org/pvldb/vol18/p3655-zhang.pdf
4. Chao et al. *STALE*. https://arxiv.org/abs/2605.06527
5. Patel. *Supersede*. https://arxiv.org/abs/2606.27472
6. Uddin et al. *From Recall to Forgetting / Memora*. https://arxiv.org/abs/2604.20006
7. Kim et al. *OAKS*. ACL 2026. https://aclanthology.org/2026.acl-long.1956/
8. Doyle. *A Truth Maintenance System*. Artificial Intelligence, 1979.
9. McAllester. Truth-maintenance / dependency reasoning work, AAAI-era literature.
10. Assumption-based TMS literature for environment/justification management.

## ConstraintAcquisitionSQL / interaction and model acquisition

1. Qiu et al. *Interactive Text-to-SQL via Expected Information Gain for Disambiguation*. https://arxiv.org/abs/2507.06467
2. PRACTIQ, NAACL 2025.
3. CLARITY, ACL Industry 2026.
4. PleaSQLarify, CHI 2026.
5. BIRD-Interact. https://bird-interact.github.io/ and associated ICLR 2026 paper.
6. Tsouros et al. *Guided Bottom-Up Interactive Constraint Acquisition*. https://arxiv.org/abs/2307.06126
7. Tsouros et al. *Learning to Learn in Interactive Constraint Acquisition*. https://arxiv.org/abs/2312.10795
8. Tsouros et al. *Generalizing Constraint Models in Constraint Acquisition (GenCon)*. https://arxiv.org/abs/2412.14950
9. Cost-sensitive / partial-feedback active-learning literature for structured prediction.
10. Value-of-information methods for deciding whether information acquisition is worth user cost.

---

# 12. Final recommendation

The literature audit changes the research program in a useful way.

Do **not** lead with SpectrumSQL: direct SQL spectrum-based fault localization predates modern LLM Text2SQL by years, while recent Text2SQL correction/verification is already test-driven and localization-aware.

Lead instead with a problem current Text2SQL systems largely assume away:

> **A SQL agent should know when the query is not the thing that is wrong.**

That gives `CertaintySQL` a crisp first paper: diagnose query fault vs data fault and return answers whose certainty is justified under inconsistent data.

The next natural layer is `TMS-SQL`:

> **A SQL agent should know when its own remembered semantic assumptions are no longer valid, and exactly which downstream query artifacts must be re-derived.**

Then `ConstraintAcquisitionSQL` supplies the learning mechanism:

> **A SQL agent should use interaction not merely to solve the current question, but to learn reusable organizational semantics that reduce future interaction.**

Together they define a coherent research agenda around **epistemically grounded database agents**: agents that acquire semantic rules, maintain their justifications under change, distinguish program errors from data inconsistencies, and expose uncertainty instead of reflexively rewriting SQL.