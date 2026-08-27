# Text2SQL Top-5 Anchored Method Ideas (2026-08-27)

> Status: literature-first research spike; no implementation changes.
> Goal: retain only method-type Text2SQL/data-agent ideas that start from a reproducible limitation in a strong recent anchor paper, define an explicit algorithmic delta, a primary quantitative metric, and a low-cost falsification test.

## Selection protocol

The retained candidates obey the following chain:

```text
anchor paper
  -> reproducible empirical claim
  -> one held-fixed limitation
  -> explicit input/intermediate/output
  -> concrete algorithm/system delta
  -> one primary metric
  -> strongest baseline
  -> 20-50 example kill test
  -> full experiment only if the phenomenon survives
```

Rejected directions include generic fault localization, generic provenance, generic TMS/memory, generic query refinement, generic agent routing, pure benchmark proposals, pure error analysis, and any direction whose contribution can be reduced to a new prompt, backbone, retriever, judge, or more candidates without a coverage/cost objective.

---

# Rank 1 — Join-Consistent Value Linker

## Anchor Paper

**DIVER: A Robust Text-to-SQL System with Dynamic Interactive Value Linking and Evidence Reasoning** — Nan et al., SIGMOD 2026.

- Paper: https://arxiv.org/abs/2602.12064
- Code: https://github.com/thatmee/DIVER
- Task: expert-free Text-to-SQL with dynamic value linking and evidence reasoning.
- Core method: iteratively identify semantic entities, probe database values, refine value-column mappings, and feed verified facts into SQL generation.
- Reported result: strong gains over prior systems, including substantially stronger challenging value-linking performance.
- Limitation used here: local value-linking candidates are scored/verified mostly per mention; multiple individually plausible mappings are not explicitly optimized as one globally consistent relational assignment.
- Collision check: value linking itself is crowded, including VLD-Bench and multiple schema/value retrieval systems. The surviving gap is not retrieval; it is constrained joint assignment over an already-fixed candidate pool.

## Task definition

```text
Input:
  NL question
  + schema/FK graph
  + top-k DIVER value-linking candidates for each mention

Intermediate object:
  factor graph over mention -> (table,column,value) assignments
  + join-connectivity, datatype, operator and path-length constraints

Output:
  globally consistent value-column assignments
  + minimal connected join subgraph
  -> unchanged DIVER SQL generator
```

## Exact Delta

| Dimension | Anchor Paper | This idea |
|---|---|---|
| Input representation | NLQ + schema + DB probe results | Same + top-k candidate pool per mention |
| Intermediate representation | verified local facts | explicit global factor graph + join subgraph |
| Training objective | none for global linker | MAP/ILP objective; optional learned pairwise factors |
| Inference | mention-wise iterative linking | constrained global assignment after local candidate generation |
| Supervision | DB observations + final SQL | gold SQL-derived linking/path labels on train split |
| Evaluation | linking F1 + EX | value-link-sensitive EX primary; F1/path recall secondary |

A V1 training-free score is:

\[
A^*=\arg\max_A \sum_m s_{local}(a_m)
+\lambda_1 C_{join}(A)
+\lambda_2 C_{type}(A,q)
+\lambda_3 C_{operator}(A,q)
-\lambda_4 L_{path}(A)
\]

subject to the selected tables admitting a legal connected join subgraph.

This is not an LLM judge: candidate retrieval, SQL generator, and DB evidence are held fixed; only the assignment algorithm changes.

## Core hypothesis

On a VLD-Bench/BIRD multi-value multi-table subset, with the same DIVER top-5 local candidate pool and the same downstream SQL generator, the proposed linker improves value-link-sensitive execution accuracy by **>=5 percentage points**, with zero additional DB calls and <=10% p50 latency increase.

## Primary metric

**Value-Link-Sensitive Execution Accuracy (VL-EX)**

\[
VL\text{-}EX=\frac1N\sum_i \mathbf{1}[Exec(\hat S_i,D_i)=Exec(S_i^*,D_i)]
\]

computed only on the pre-registered value-link-sensitive subset.

## Secondary metrics

- Value Linking F1
- Join-path Recall
- extra DB calls
- p50 latency

## Strongest baselines

1. DIVER local linking
2. DIVER local top-1 + shortest-FK-path heuristic
3. CodeS-style coarse-to-fine value linking
4. XiYanSQL/M-Schema-style schema/value representation

## Minimal 20–50 example test

Select **40** cases satisfying:

- gold SQL uses >=2 tables;
- >=2 value mentions, or one mention maps to multiple plausible columns;
- the gold mapping exists in DIVER top-5 local candidates.

Compare:

```text
A independent local top-1
B local top-1 + shortest-path heuristic
C global constrained inference
```

Success: C solves at least 2 additional tasks out of 40 and improves assignment accuracy over B.

Kill if:

- local Recall@5 <80% (retrieval remains the true bottleneck);
- shortest-path heuristic is within 1 case of the proposed method;
- linking F1 rises but EX does not;
- effect appears only on one database.

## Full experiment

Datasets:

- VLD-Bench
- BIRD-dev pre-registered multi-value/multi-table subset

Report paired bootstrap 95% CI by database and McNemar significance on EX.

## Ablations

1. remove join-connectivity factor;
2. remove datatype/operator factor;
3. replace global MAP with independent top-1.

## Final score

- Anchor Strength: 5/5
- Quantifiability: 5/5
- Novelty: 4/5
- Feasibility: 5/5
- Reproduction Cost: 2/5
- Incremental-Risk: 3/5

**Two-sentence pitch.** DIVER solves whether each value mention has a plausible local mapping, but does not explicitly solve whether several mappings jointly form a legal relational interpretation. Freeze retrieval and SQL generation, and improve only global assignment under relational constraints.

---

# Rank 2 — Semantic-Coverage Candidate Resampler

## Anchor Paper

**R3-SQL: Ranking Reward and Resampling for Text-to-SQL** — Han et al., Findings of ACL 2026.

- Paper: https://aclanthology.org/2026.findings-acl.2146/
- Code: https://github.com/ldilab/R3_SQL
- Task: improve candidate generation/selection through ranking reward and selective resampling.
- Core result: selective resampling increases candidate upper-bound coverage and improves final EX; always-resampling has little benefit.
- Limitation used here: resampling is not explicitly targeted at missing semantic factors, so many new candidates may be syntactic variants of the same wrong interpretation.
- Collision check: DivSkill-SQL already addresses complementary skills/correlated failures, so the surviving claim must be fixed-budget per-instance **semantic-factor coverage**, not generic diversity.

## Task definition

```text
Input:
  question + schema + K SQL candidates

Intermediate object:
  semantic signature per candidate:
  {tables, join_path, result_grain, measure/aggregate,
   predicates, temporal column/window, distinct/set semantics}

Output:
  a replacement/resampling target that fills an uncovered semantic region
  -> fixed-size candidate pool
  -> unchanged R3-SQL ranker
```

## Exact Delta

| Dimension | Anchor Paper | This idea |
|---|---|---|
| Input representation | raw SQL candidates | candidates + semantic signatures |
| Intermediate representation | execution-equivalence groups | execution groups + semantic-factor coverage matrix |
| Training objective | ranking + resample trigger | ranking unchanged; optimize uncovered plausible signatures |
| Inference | audit -> free resampling | audit -> identify semantic gaps -> constrained replacement |
| Supervision | correctness/ranking reward | train-only gold signatures for plausibility estimation |
| Evaluation | EX / candidate recall | OracleEX@K primary; Top-1 EX secondary |

Candidate signature:

\[
z(s)=(T,J,G,A,F,\tau,D)
\]

Target semantic region:

\[
z^*=\arg\max_z P(z|q,S)\left(1-\max_{z_i\in C} sim(z,z_i)\right)
\]

Candidate count and generation-call budget remain fixed.

## Core hypothesis

On BIRD-dev and Spider-DK, with the same generator, K=8 and the same generation-call budget, semantic-targeted resampling improves **Oracle EX@8 by >=4pp**, final Top-1 EX by >=1.5pp, and uses <=1.1x generated tokens relative to R3-SQL resampling.

## Primary metric

**Oracle Execution Accuracy@K**

\[
OracleEX@K=\frac1N\sum_i \mathbf{1}[\exists s\in C_i^K: Exec(s,D_i)=Exec(S_i^*,D_i)]
\]

## Secondary metrics

- Top-1 EX using unchanged R3 ranker
- semantic-signature coverage
- generated tokens
- resampling trigger rate

## Strongest baselines

1. R3-SQL original selective resampling
2. always-temperature resampling
3. DivSkill-style complementary prompting
4. random semantic target

## Minimal 20–50 example test

Select **40** BIRD-dev cases with initial OracleEX@8=0 and R3 audit triggering resampling.

Each method gets exactly eight additional generation calls.

Success: targeted resampling produces at least two more newly-correct-covered cases than R3 free resampling.

Kill if:

- <=1 extra correct candidate;
- random prompt variation matches the method;
- OracleEX rises but unchanged R3 Top-1 never improves;
- benefit disappears at fixed candidate count;
- residual method is not materially distinct from DivSkill-SQL.

## Full experiment

Datasets:

- BIRD-dev
- Spider-DK
- optional OOD: EHRSQL

Use database-cluster bootstrap, >=4 candidate-generation seeds, and fixed total generation budget.

## Ablations

1. remove join/grain factors;
2. replace semantic coverage with AST diversity;
3. keep candidates fixed and disable targeted replacement.

## Final score

- Anchor Strength: 5/5
- Quantifiability: 5/5
- Novelty: 4/5
- Feasibility: 4/5
- Reproduction Cost: 2/5
- Incremental-Risk: 3/5

**Two-sentence pitch.** R3-SQL proves bounded candidate recall is a real bottleneck, but stochastic resampling need not explore a new semantic interpretation. Under exactly the same K and generation cost, explicitly fill uncovered semantic factors and measure OracleEX@K.

---

# Rank 3 — Interaction-Aware Clause Reward

## Anchor Paper

**EXPO-SQL: Execution-based Clause-level Policy Optimization for Text-to-SQL** — Lee et al., Findings of ACL 2026.

- Paper: https://aclanthology.org/2026.findings-acl.1107/
- Code: https://github.com/jhn25/EXPO-SQL
- Task: RL fine-tuning for Text-to-SQL using execution-based clause-level rewards.
- Core result: clause-level execution reward outperforms query-level PPO/GRPO-style feedback, with especially larger gains on challenging queries.
- Limitation used here: credit is primarily unary clause attribution, while SQL semantics often contain non-additive interactions such as JOIN x WHERE, SELECT x GROUP BY, JOIN x DISTINCT, ORDER BY x LIMIT.
- Collision check: FINER-SQL and ReEx-SQL already provide fine-grained/execution-aware rewards; the surviving contribution is specifically **interaction credit**, not additional execution feedback or fault localization.

## Task definition

```text
Input:
  question + schema + gold SQL + policy-sampled SQL

Intermediate object:
  clause interaction graph
  + unary intervention gain
  + pairwise intervention synergy

Output:
  interaction-aware clause reward
  -> GRPO policy update
```

## Exact Delta

| Dimension | Anchor Paper | This idea |
|---|---|---|
| Input representation | sampled SQL + gold execution | same + clause-pair counterfactual variants |
| Intermediate representation | C_err clause set | clause graph + unary/pairwise synergy |
| Training objective | clause-weighted GRPO | interaction-credit-weighted GRPO |
| Inference | ordinary generation | unchanged; no new inference-time module |
| Supervision | incremental execution discrepancy | counterfactual gold-clause replacement execution |
| Evaluation | EX | BIRD Challenging EX primary |

For clause i:

\[
u_i=Score(Exec(y^{i\leftarrow gold}))-Score(Exec(y))
\]

For adjacent pair i,j:

\[
u_{ij}=Score(Exec(y^{i,j\leftarrow gold}))-Score(Exec(y))
\]

Interaction:

\[
I_{ij}=\max(0,u_{ij}-u_i-u_j)
\]

Reward:

\[
r_i=r_i^{EXPO}+\lambda\sum_{j\in N(i)}I_{ij}/2
\]

Only a small preselected set of structurally adjacent pairs is evaluated.

## Core hypothesis

With the same 7B backbone, training data, rollout count and inference budget, interaction-aware reward improves **BIRD challenging EX by >=3pp**, overall EX by >=1.5pp, while increasing training DB executions by <=1.25x relative to EXPO-SQL.

## Primary metric

**BIRD Challenging Execution Accuracy**.

## Secondary metrics

- overall BIRD EX
- Spider-DK EX
- training DB executions per rollout
- inference token cost

## Strongest baselines

1. query-level GRPO
2. Arctic-SQL-R1
3. EXPO-SQL
4. ReEx-SQL
5. FINER-SQL where reproducible

## Minimal 20–50 example test

Before training, sample **40** EXPO failure cases with at least two candidate error clauses.

Compute unary and top-3 structurally adjacent pair counterfactual interventions.

Define:

\[
InteractionOnlyRate=P(u_i=0,u_j=0,u_{ij}>0)
\]

Success:

- InteractionOnlyRate >=15%; and
- pairwise signal improves correct credit ordering by >=10pp over unary attribution.

Kill if:

- InteractionOnlyRate <8%;
- unary EXPO already captures most pair signal;
- gains are explainable only by extra DB executions;
- challenging EX gain <2pp in a pilot train run;
- same-budget FINER/ReEx matches the method.

## Full experiment

Datasets:

- BIRD
- Spider-DK / Spider-Syn

Use >=3 training seeds, paired bootstrap 95% CI, and McNemar test.

## Ablations

1. lambda=0 -> EXPO;
2. only JOIN-WHERE pair interactions;
3. shuffled pair-interaction labels.

## Final score

- Anchor Strength: 5/5
- Quantifiability: 5/5
- Novelty: 4/5
- Feasibility: 3/5
- Reproduction Cost: 5/5
- Incremental-Risk: 3/5

**Two-sentence pitch.** EXPO-SQL establishes the value of clause-level credit, but approximates correctness as mostly clause-wise. Counterfactual pair interventions detect genuinely non-additive SQL semantics and alter only the training reward, not the inference scaffold.

---

# Rank 4 — Workload-Aware Offline SQL Explorer

## Anchor Paper

**SQLAgent: Learning to Explore Before Generating as a Data Engineer** — Jiang et al., Findings of ACL 2026.

- Paper: https://aclanthology.org/2026.findings-acl.1959/
- Code: https://github.com/Westlake-AGI-Lab/SQLAgent
- Task: persistent database-specific offline exploration before query-time SQL generation.
- Core result: database exploration can be amortized across later queries and improves BIRD/Spider2 performance.
- Limitation used here: exploration terminates around broad field/schema coverage (90% fields or 300 attempts) rather than optimizing expected value under the actual future workload distribution.
- Collision check: AgentSM, Memo-SQL and memory-crystallization make reusable memory crowded. This idea therefore changes the **offline acquisition policy**, not the memory representation.

## Task definition

```text
Input:
  database schema graph
  + unlabeled historical NL-question stream
  + fixed exploration budget B

Intermediate object:
  schema-region demand distribution
  + SQLAgent exploration frontier

Output:
  B validated (schema, SQL, NL-description) exploration artifacts
  -> unchanged SQLAgent deployment stage
```

## Exact Delta

| Dimension | Anchor Paper | This idea |
|---|---|---|
| Input representation | DB schema graph | schema graph + unlabeled workload questions |
| Intermediate representation | exploration tree | tree + region demand mass |
| Training objective | broad familiarity/coverage | maximize held-out workload utility at fixed B |
| Inference | explore until coverage/attempt threshold | hard B; priority weighted by workload demand |
| Supervision | validation feedback | same validation; no historical gold SQL |
| Evaluation | EX + amortized cost | Held-out EX@fixed initialization budget |

Priority:

\[
U(v)=Q_{valid}(v)+\alpha Novelty(v)+\beta DemandMass(v|W)-\gamma Cost(v)
\]

with:

\[
DemandMass(v|W)=\frac1{|W|}\sum_{q\in W}P(v\text{ relevant}|q)
\]

## Core hypothesis

On persistent BIRD database splits, with B=100 exploration attempts and the same backbone, workload-aware exploration improves **held-out EX@B by >=4pp** relative to coverage-driven SQLAgent exploration, without increasing initialization tokens and with <=1.05x per-query tokens.

## Primary metric

**Held-out Execution Accuracy at fixed initialization budget B (EX@B).**

## Secondary metrics

- initialization tokens
- per-query tokens
- schema field coverage
- first-attempt EX

## Strongest baselines

1. SQLAgent original coverage-driven exploration
2. random schema-region exploration
3. schema-centrality exploration
4. retrieval-score/workload-frequency heuristic
5. query-time SDE/APEX as a cost reference

## Minimal 20–50 example test

Choose 2–4 BIRD databases with enough questions; split each into unlabeled historical question text and **40–50 held-out questions**.

Compare original SQLAgent, random, centrality, and workload-aware exploration with B=50.

Success: >=2 extra correct answers out of 50 with equal or lower initialization tokens.

Kill if:

- shuffled historical workload performs the same;
- coverage baseline already saturates;
- EX gain <=2pp;
- method needs gold SQL/schema usage leakage;
- effect appears only on one database.

## Full experiment

Datasets:

- BIRD within-database historical/held-out split
- Spider within-database split

Hyperparameters are tuned on some databases and transferred to unseen databases, where only unlabeled question streams are visible.

Report cumulative token-cost amortization curves versus query count.

## Ablations

1. remove workload term;
2. shuffle historical questions;
3. keep workload term, remove novelty term.

## Final score

- Anchor Strength: 4/5
- Quantifiability: 5/5
- Novelty: 4/5
- Feasibility: 4/5
- Reproduction Cost: 3/5
- Incremental-Risk: 3/5

**Two-sentence pitch.** SQLAgent spends offline exploration budget toward broad schema coverage, but production workloads are non-uniform. Keep memory and deployment unchanged; optimize only where the fixed precomputation budget is spent.

---

# Rank 5 — Dependency-Aware Parallel SQL Test Scheduler

## Anchor Paper

**PExA: Parallel Exploration Agent for Complex Text-to-SQL** — Parekh et al., ACL 2026 Short.

- Paper: https://aclanthology.org/2026.acl-short.48/
- Task: lower wall-clock cost of complex Text-to-SQL exploration by decomposing reasoning into independent, self-contained SQL test cases and running them in parallel.
- Core result: parallel tests improve the performance/latency tradeoff on Spider2-style workloads.
- Limitation used here: independence is a design constraint; some useful tests require a prior observation to instantiate a later test.
- Collision check: SDE-SQL/APEX/ReEx already cover generic sequential/adaptive exploration. The surviving problem is **typed dependency scheduling under fixed concurrency and fixed total test budget**.

## Task definition

```text
Input:
  question + schema
  + PExA-style generated test intents
  + concurrency width W

Intermediate object:
  typed test DAG
  each test has requires={slots}, produces={slots}

Output:
  scheduled batches of tests
  + collected typed facts
  -> unchanged PExA proposer/final synthesizer
```

## Exact Delta

| Dimension | Anchor Paper | This idea |
|---|---|---|
| Input representation | independent test cases | typed test intents with prerequisites/products |
| Intermediate representation | flat parallel set | dependency DAG |
| Training objective | none | none; deterministic scheduling objective |
| Inference | run all independent tests in parallel | schedule ready antichains under W |
| Supervision | execution results | execution results converted to typed slot values |
| Evaluation | EX + wall time | latency-constrained EX primary |

Scheduling objective:

\[
schedule=\arg\min Makespan(G,W)
\]

subject to:

\[
pred(t)\subseteq completed
\]

Ready-node priority can combine critical path and expected evidence coverage.

## Core hypothesis

On Spider2-Snow, with max concurrency W=6 and the same total test-call budget, dependency-aware scheduling improves **Latency-Constrained EX@300s by >=4pp** over flat PExA, while plain EX drops by no more than 1pp and total LLM/DB calls stay <=1.05x.

## Primary metric

**Latency-Constrained Execution Accuracy @300s**

\[
LC\text{-}EX@300=\frac1N\sum_i\mathbf{1}[correct_i \land latency_i\le300s]
\]

## Secondary metrics

- plain EX
- p95 latency
- DB calls
- LLM calls

## Strongest baselines

1. PExA flat parallel
2. same tests fully sequential
3. ReFoRCE
4. SDE-SQL/APEX-SQL
5. random DAG-edge control

## Minimal 20–50 example test

Select **30** PExA hard cases with >=4 generated tests.

Annotate whether a test truly requires information produced by another test.

Compare flat parallel, all-sequential, and typed-DAG scheduling with identical test budget.

Success:

- >=20% tasks contain a real dependency;
- DAG scheduling solves >=1 extra task out of 30;
- latency <=1.1x flat PExA.

Kill if:

- dependency incidence <10%;
- prerequisites can be eliminated by schema prompt alone;
- critical path destroys the parallelism advantage;
- gains come only from more tests;
- ordinary sequential APEX/SDE dominates at equal wall time.

## Full experiment

Datasets:

- Spider2-Snow
- BIRD challenging

Use >=4 inference seeds and paired bootstrap 95% CI. PExA code availability is weaker than the first four anchors, so reproduction risk is explicitly higher.

## Ablations

1. remove dependency edges;
2. preserve DAG but randomize ready-node scheduling;
3. unlimited concurrency to separate dependency benefit from scheduling benefit.

## Final score

- Anchor Strength: 5/5
- Quantifiability: 5/5
- Novelty: 3/5
- Feasibility: 3/5
- Reproduction Cost: 4/5
- Incremental-Risk: 4/5

**Two-sentence pitch.** PExA obtains low latency by requiring all tests to be independent, excluding diagnostic actions that depend on earlier database observations. Compile test intents into a typed dependency DAG and optimize latency-constrained execution under the same concurrency and test budget.

---

# Portfolio recommendation

The next machine experiments should not implement the full methods immediately. Run the five pre-registered kill tests in this order:

1. **Join-Consistent Value Linker** — verify that gold local candidates exist but independent assignment combines them incorrectly.
2. **Semantic-Coverage Candidate Resampler** — verify fixed-K targeted resampling raises OracleEX@K beyond R3/DivSkill-style baselines.
3. **Interaction-Aware Clause Reward** — verify pairwise-only clause credit is common enough before any RL retraining.
4. **Workload-Aware Offline SQL Explorer** — verify real workload text predicts where exploration budget should be spent.
5. **Dependency-Aware Parallel SQL Test Scheduler** — verify prerequisite test chains are frequent enough to justify a DAG scheduler.

Any candidate that fails its kill criterion should be retired rather than rescued by adding modules.