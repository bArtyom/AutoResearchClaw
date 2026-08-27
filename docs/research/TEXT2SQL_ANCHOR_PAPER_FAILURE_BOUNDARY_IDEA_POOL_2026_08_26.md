# Text2SQL Anchor-Paper Failure-Boundary Idea Pool (2026-08-26)

> Status: literature-first research spike; no implementation changes.
> Goal: generate new Text2SQL / data-agent ideas from verified failure boundaries of strong recent papers, rather than starting from a mechanism name.
> Process: anchor paper -> reproduce phenomenon -> isolate one boundary -> cheap falsification -> only then formulate a method.

---

## 1. Research rule for this pass

This pass deliberately rejects the pattern:

```text
method name -> search for a Text2SQL use case -> add an agent/module -> benchmark
```

Instead, each idea must satisfy:

1. start from a recent strong paper with public code/data when possible;
2. identify the paper's core empirical claim;
3. identify one concrete assumption or untested boundary;
4. define an experiment that can fail cheaply on 20-50 examples;
5. avoid requiring a synthetic benchmark unless the scientific claim is explicitly about controlled counterfactuals;
6. use an existing benchmark/environment whenever possible;
7. set a stop condition before proposing a full system.

The primary anchors in this pass are:

- SDE-SQL, ACL 2026;
- DPC, ACL 2026;
- BIRD-INTERACT, ICLR 2026 Oral;
- DeepEye-SQL, SIGMOD 2026;
- Spider 2.0-AIFunc, 2026 benchmark with public code/data.

Secondary evidence comes from R3-SQL, CHASE-SQL, MIRA, GATE, semantic-layer benchmarks, the VLDB 2026 benchmark-correction study, and context-length / schema-representation work.

---

# 2. Anchor evidence cards

## Anchor A — SDE-SQL (ACL 2026)

### Core problem

Static schema/context is insufficient for understanding real database content.

### Core hypothesis

Allowing the model to generate and execute SQL probes during inference improves Text2SQL.

### Evidence

SDE-SQL reports an 8.02% relative execution-accuracy improvement over vanilla Qwen2.5-72B-Instruct on BIRD, with public code for BIRD/Spider.

Reference:
https://aclanthology.org/2026.acl-long.116/
https://github.com/Lancelot-Xie/SDE-SQL

### Boundary not directly answered

The paper establishes that *access to active exploration* helps. It does not establish:

- how much of the collected evidence is actually necessary;
- whether correct evidence can become harmful when accumulated;
- whether the order in which observations arrive changes the final semantic interpretation;
- whether the performance gain comes from a few decisive probes or broad exploration.

### Reproduction first

Run the released SDE-SQL pipeline on a 20-50 task slice where probing changes the final answer relative to vanilla generation. Preserve complete trajectories.

---

## Anchor B — DPC (ACL 2026)

### Core problem

Text2SQL often has a large Pass@K vs Pass@1 gap: a correct SQL exists among candidates, but the selector chooses the wrong one.

### Core hypothesis

Cross-paradigm verification on a Minimal Distinguishing Database can select candidates better than same-model voting/judging.

### Evidence

DPC uses SQL candidates, a synthetic distinguishing micro-database, and an independently generated Python/Pandas solution. It reports up to +2.2 absolute accuracy over strong training-free selectors on BIRD/Spider, with public code.

Reference:
https://aclanthology.org/2026.acl-long.313/
https://github.com/HKUSTDial/DPC

### Boundary not directly answered

Selection cannot recover if every candidate shares the same wrong *semantic commitment*. DPC changes the verification paradigm, but the SQL and Python programs may still share the same upstream interpretation of:

- entity;
- metric;
- time scope;
- join path;
- aggregation grain;
- business rule.

### Reproduction first

Reproduce DPC on 20-50 BIRD examples with multiple candidate SQLs. For each candidate set, extract semantic commitments before testing any new selector.

---

## Anchor C — BIRD-INTERACT (ICLR 2026 Oral)

### Core problem

Real database assistants require dynamic interaction, clarification, exploration, error recovery and CRUD, rather than one-shot SQL generation.

### Core hypothesis

An interactive environment with a user simulator, hierarchical knowledge, metadata and database tools exposes a much harder and more realistic problem than static Text2SQL.

### Evidence

BIRD-INTERACT provides 600 tasks, a Lite subset, user simulator, DB environment and ADK-based modular implementation. Reported GPT-5 success is 8.67% in c-Interact and 17.00% in a-Interact on the full suite.

Reference:
https://proceedings.iclr.cc/paper_files/paper/2026/hash/496b549556509bbb9770bf9d335c5800-Abstract-Conference.html
https://github.com/bird-bench/BIRD-Interact

### Boundary not directly answered

The benchmark mixes several information sources:

- user answers;
- database observations;
- hierarchical KB;
- metadata/documentation;
- prior dialogue state.

It is therefore not obvious how much *human interaction is irreducibly necessary*, versus substitutable by machine-readable evidence.

### Reproduction first

Select 20-30 BIRD-INTERACT-Lite tasks with at least one clarification/user interaction. Audit whether the information returned by the simulator exists elsewhere in the DB/KB/metadata.

---

## Anchor D — DeepEye-SQL (SIGMOD 2026)

### Core problem

Reliability failures are system-level, not only generation-level.

### Core hypothesis

A software-engineering-style pipeline with grounding, N-version generation, deterministic checks/revision and confidence-aware selection improves Text2SQL reliability.

### Evidence

DeepEye-SQL reports 73.5% BIRD-Dev and 89.8% Spider-Test execution accuracy in the paper; its public repository includes staged pipeline code, snapshots, outputs and runbooks for BIRD/Spider/Spider2.

Reference:
https://arxiv.org/abs/2510.17586
https://github.com/HKUSTDial/DeepEye-SQL

### Boundary not directly answered

End-to-end ablation tells us whether removing a component hurts *on average*, but not:

- where the first wrong semantic commitment entered the pipeline;
- whether later stages corrected or merely preserved it;
- which stages introduce new errors on originally correct intermediate states.

### Reproduction first

Use released intermediate snapshots on a small set of successes and failures. Track table choices, joins, filters, aggregation and revision decisions stage by stage.

---

## Anchor E — Spider 2.0-AIFunc (2026)

### Core problem

Cloud databases now expose LLM-backed AI functions inside SQL, creating a new AI-native query language regime.

### Core hypothesis

Models can be evaluated on generating SQL that correctly invokes AI_CLASSIFY, AI_FILTER, AI_AGG, AI_EXTRACT, AI_SIMILARITY and AI_SENTIMENT within enterprise workflows.

### Evidence

Spider 2.0-AIFunc contains hundreds of verified tasks across real databases. Strong proprietary models reach roughly 67-70% execution accuracy. Crucially, function-selection accuracy is already extremely high for strong models because the benchmark intentionally makes intended AI-function use explicit and audits away decorative AI use.

Reference:
https://arxiv.org/abs/2607.06229
https://github.com/Leolty/Spider2-AIFunc

### Boundary not directly answered

The benchmark tests *how to use the intended AI function*, not the harder planning question:

> Should this user request use an AI operator at all, and if so which semantic operator is actually necessary?

### Reproduction first

Take 20-50 public AIFunc tasks and inspect how strongly the instruction reveals the function type/parameters. Compare with ordinary Spider2 tasks where deterministic SQL suffices.

---

# 3. New idea pool derived from failure boundaries

## Idea 1 — SemCov-SQL: candidate selection has a semantic-coverage ceiling

### Problem first

DPC, R3-SQL, CHASE-SQL and other systems invest heavily in candidate ranking/selection. But no selector can recover a correct interpretation that was never generated.

The real latent variable may be *semantic candidate coverage*, not candidate count or surface diversity.

### Scientific object

Represent each candidate by semantic commitments such as:

```text
entity set
join graph
measure definition
aggregation grain
time interpretation
filters
set/bag choice
null policy
```

Define semantic coverage of a candidate pool and compare it with Pass@K.

### Hypothesis

After controlling for candidate count and AST/token diversity, semantic-commitment coverage explains most of the remaining variance in oracle Pass@K.

### Cheap falsification

20-50 BIRD examples; generate 8-16 candidates each using existing generators. Cluster candidates by semantic commitment vector.

Stop if semantic coverage adds little predictive value over ordinary candidate count / SQL structural diversity.

### If it survives

Only then design a generator that explicitly fills missing semantic alternatives.

### Collision risk

CHASE-SQL/XiYan-SQL already pursue diverse candidate generation; R3-SQL recognizes candidate recall. The surviving distinction is *measuring and optimizing diversity in the space of semantic commitments rather than generation styles or execution groups*.

---

## Idea 2 — Shared-Blind-Spot SQL: cross-paradigm agreement can still be jointly wrong

### Problem first

DPC assumes SQL and Python/Pandas have sufficiently different failure modes. But both are generated from the same natural-language interpretation.

### Hypothesis

A non-trivial fraction of DPC errors are not implementation errors but *shared upstream semantic misunderstandings*: SQL and Python agree because both misread the task in the same way.

### Cheap falsification

Take DPC failures. Manually annotate 30-50 examples into:

```text
implementation disagreement
MDD construction failure
shared NL interpretation error
schema grounding error
annotation/evaluation issue
```

Stop if shared semantic misunderstanding is rare (<10% of DPC residual errors).

### If it survives

The next paper should target independent *semantic interpretation generation* before cross-paradigm execution, not add another verifier.

---

## Idea 3 — EvidenceSufficiency-SQL: how much database exploration was actually necessary?

### Problem first

SDE-SQL proves exploration access helps, but active agents can issue many probes. The field lacks a notion of the *minimal sufficient evidence set* for a Text2SQL decision.

### Task

Given a successful exploration trajectory, find the smallest subset of observed database facts that preserves the correct final SQL.

### Metrics

```text
minimal evidence count
minimal evidence tokens
redundant probe fraction
accuracy vs evidence retained
```

### Cheap falsification

Replay 20-50 successful SDE-SQL trajectories while deleting observations. Start with greedy deletion / leave-one-probe-out rather than training anything.

Stop if almost all probes are necessary or trajectory replay is too unstable to assign evidence sufficiency.

### Why it matters

If 70-90% of probes are unnecessary, the problem becomes evidence acquisition efficiency with a directly measured target rather than vague "better exploration".

---

## Idea 4 — Evidence Saturation Reversal: can more correct evidence reduce SQL accuracy?

### Problem first

General long-context research shows performance can decline purely with context length even when relevant information is retrievable. Active SQL agents continuously append valid observations.

### Hypothesis

Text2SQL has an evidence-saturation point beyond which additional *correct but non-decisive* database observations reduce final accuracy.

### Cheap falsification

For SDE-SQL tasks, hold the decisive evidence fixed and progressively append other genuine probe outputs from the same database.

Measure accuracy as a function of observation count/tokens.

Stop if performance is monotonic/non-decreasing across models.

### Novelty boundary

Not generic context-window research: the independent variable is executable relational evidence gathered by the agent during task solving.

---

## Idea 5 — Evidence Order Invariance for Data Agents

### Problem first

If two observations are both factual and logically independent, a rational evidence accumulator should not radically change its conclusion only because their order changed.

### Hypothesis

Current database agents exhibit high path dependence: permuting the same set of tool observations changes schema choices and final SQL.

### Cheap falsification

Take completed SDE-SQL/APEX-style trajectories and replay the same observation set under random valid orderings.

Primary metric:

```text
same-evidence answer disagreement rate
```

Stop if disagreement is negligible.

### Value

A positive result creates a clean target for evidence-state representations that are more stable than raw dialogue concatenation.

---

## Idea 6 — Irreducible Interaction Rate: how many user turns are fundamentally necessary?

### Problem first

BIRD-INTERACT shows interaction matters, but the environment exposes user answers, KB, metadata and DB tools simultaneously. A clarification turn may be substitutable by machine-readable evidence.

### Scientific question

For a task, what is the minimum amount of information that must come from the user rather than from the database/metadata/KB?

### Cheap falsification

Audit 20-30 Lite tasks with clarification turns. For each user answer, label:

```text
USER_ONLY
DB_DERIVABLE
KB_DERIVABLE
METADATA_DERIVABLE
MULTI_SOURCE_DERIVABLE
```

Stop if almost all required information is genuinely user-only.

### If it survives

Define `Irreducible Human Turns` and `Avoidable Human Turns`; then evaluate agents by success at a fixed irreducible-user budget.

### Why this is different

This does not propose another ask/act policy. It asks whether the benchmark's "interaction" difficulty is actually a dialogue problem or an information-source-selection problem.

---

## Idea 7 — Interaction Failure Decomposition: asking, integrating, or acting?

### Problem first

A failed interactive trajectory can result from at least three distinct failures:

1. asking the wrong question;
2. receiving useful information but failing to integrate it;
3. correctly updating intent but generating/executing the wrong SQL/action.

Current end-to-end success conflates them.

### Cheap falsification

Use BIRD-INTERACT trajectories on 30-50 failed tasks. Replace one stage at a time with oracle information:

```text
oracle clarification choice
oracle user answer
oracle updated intent
oracle SQL/action
```

Measure recovery.

### Stop condition

If no single stage explains a meaningful share of failures, do not build a specialized component.

### Potential paper

A boundary/diagnostic paper identifying the dominant bottleneck in interactive Text2SQL rather than adding a broad new agent.

---

## Idea 8 — Semantic First-Error Attribution in SQL pipelines

### Problem first

DeepEye-SQL demonstrates that orchestration helps, but global ablations do not tell us where wrong semantics first enter the workflow.

### Task

For each failed instance, identify the earliest stage at which the trajectory becomes incompatible with the gold task semantics.

Possible states:

```text
correct grounding -> wrong schema link
correct schema -> wrong join
correct plan -> wrong generation
correct candidate -> harmful revision
correct candidate set -> wrong selection
```

### Cheap falsification

Audit 30-50 DeepEye failures using released pipeline snapshots.

Stop if intermediate states are too opaque to assign first-error labels reliably.

### Value

If 60%+ of final errors originate before generation, yet most research optimizes correction/selection, that is a strong research-prioritization result.

---

## Idea 9 — Error Stickiness: which semantic mistakes survive every downstream verifier?

### Problem first

A complex pipeline may repeatedly operate on an early wrong assumption instead of revisiting it.

### Hypothesis

Certain semantic errors—especially source-table choice, business metric choice or aggregation grain—become "sticky" and survive revision/verification far more often than syntax/local predicate errors.

### Cheap falsification

On DeepEye or another staged system, construct an error transition matrix:

```text
error type at stage t -> fixed / unchanged / transformed at stage t+1
```

Stop if persistence rates do not differ meaningfully by error type.

### If it survives

Future methods should explicitly reopen high-stickiness commitments rather than globally increasing correction depth.

---

## Idea 10 — Reliability Tax: how often does a reliability stage damage a correct intermediate answer?

### Problem first

Pipeline components are usually justified by average net gain. But a reliability component can both fix and corrupt answers.

### Metric

For each stage:

```text
repair_gain = wrong -> correct
regression_tax = correct -> wrong
net_gain = repair_gain - regression_tax
```

### Cheap falsification

Use DeepEye snapshots and MIRA-style correct-query preservation analysis on 30-50 examples.

Stop if regression tax is near zero for all stages.

### Collision warning

MIRA explicitly studies correct-query corruption in memory-based correction, so novelty requires a *stagewise systems-level reliability accounting* rather than another SQL corrector.

---

## Idea 11 — AI-or-Not-SQL: operator necessity before AI-native SQL generation

### Problem first

Spider2-AIFunc intentionally makes AI-function usage explicit and reports near-perfect function-selection accuracy for strong models. It therefore does not test the harder planning decision:

> Is an AI operator necessary for this request at all?

### New task object

Given an enterprise analytical request and available relational/AI operators, predict one of:

```text
DETERMINISTIC_SQL_SUFFICIENT
AI_OPERATOR_REQUIRED
EITHER_WITH_TRADEOFF
INSUFFICIENT_SPECIFICATION
```

### Cheap falsification

Build a manually audited 40-60 task pilot from ordinary Spider2-Snow + AIFunc tasks. Do not change database contents. Remove explicit function-name hints only where human reviewers agree the intent remains clear.

Stop if strong models already classify operator necessity at >95% accuracy without scaffolding.

### Why it is deeper than function selection

It changes the planning object from "generate this AI function correctly" to "choose the computational semantics required by the user intent".

---

## Idea 12 — Implicit AI-Intent Benchmark: AIFunc without function-name leakage

### Problem first

Spider2-AIFunc deliberately refines instructions so the intended AI function and parameters are explicit. This is necessary for benchmark determinism but makes function choice relatively easy.

### Hypothesis

When function names/label schemas are not explicitly given, model performance collapses primarily at *semantic operator specification*, not SQL syntax.

### Cheap falsification

Human-rewrite 30-50 tasks so the requested semantic operation remains clear but platform-specific function names are removed.

Example:

```text
explicit:
classify reviews into ['durability','price','delivery'] using AI_CLASSIFY

implicit:
for each review, assign the most appropriate issue category among durability, price and delivery
```

Stop if degradation is small.

### Contribution boundary

This is not "another AIFunc benchmark" unless the pilot reveals a large, reproducible operator-specification gap.

---

## Idea 13 — Semantic Annotation ROI: what should an organization document first?

### Problem first

Semantic-layer studies show very large accuracy gains from small hand-authored business-context documents. But production teams have limited expert annotation time.

The missing question is not "does a semantic layer help?" but:

> Which semantic facts are worth documenting first under a fixed budget?

### Existing support

The 2026 semantic-layer benchmark publicly releases a 4 KB semantic document and 100 paired questions. Another enterprise semantic-layer agent explicitly discusses semantic-layer quality and overfitting.

### Cheap falsification

Split the semantic document into atomic facts. On 20-50 questions, perform leave-one-fact-out / small-subset ablations.

Measure:

```text
accuracy gain per semantic fact
accuracy gain per 100 tokens
coverage across question families
```

Stop if gains are uniformly distributed or require nearly the full document.

### If it survives

Formulate budgeted semantic-layer construction as a workload-driven documentation problem, not an LLM prompt problem.

---

## Idea 14 — Semantic Layer Minimality: is 4 KB of context actually 400 bytes of decisive semantics?

### Problem first

Large gains from semantic layers do not reveal how much of the document is causal.

### Hypothesis

A small subset of business rules accounts for most of the accuracy gain, while the rest acts as redundant context.

### Cheap falsification

Use the open semantic-layer benchmark; find a minimal fact subset that preserves 90-95% of the full-layer gain.

Stop if the layer is not compressible without large loss.

### Distinction from schema compression

DBCC compresses large database context structurally. This idea asks about *causal minimality of business semantics* in a curated semantic layer.

---

## Idea 15 — Semantic-Layer Generalization Audit: knowledge or benchmark leakage?

### Problem first

A semantic layer can help because it captures reusable domain semantics—or because it was authored with benchmark questions in mind.

### Testable boundary

Hold out complete semantic concepts or workload families when authoring/deriving the layer, then test whether the remaining layer helps genuinely unseen questions.

### Cheap falsification

On the open paired benchmark, partition questions by business concept (revenue, policy, customer status, time convention, etc.). Rebuild or prune the semantic layer without held-out-family-specific wording.

Stop if the original gain transfers cleanly to concept-held-out evaluation.

### Why this matters

If gains vanish under concept holdout, "semantic layer quality" may be closer to workload-specific programming than general organization knowledge.

---

## Idea 16 — Is SQL Memory just a latent semantic layer?

### Problem first

Memory-crystallization work shows that database-specific corrected queries improve future first-attempt accuracy and that database-specific content is the main operating ingredient. GATE also accumulates execution-grounded reusable grounding knowledge.

This raises a mechanistic question:

> Are episodic Text2SQL memories actually functioning as an implicit semantic layer?

### Cheap falsification

Take verified same-database memory episodes and create three equal-token conditions:

```text
raw episodic memories
fact-only distilled database semantics
SQL-pattern-only summaries
no memory
```

Evaluate held-out same-database first-attempt accuracy.

Stop if fact-only distillation clearly underperforms raw episodes.

### Value

A positive result would simplify the research object from "memory architecture" to "automatic semantic-layer induction" and explain why database-specific content dominates.

### Collision warning

GATE already bootstraps reusable semantic groundings from execution. A publishable contribution would need controlled mechanistic evidence that episodic-memory gains can be explained/compressed by declarative semantics, not merely another memory-to-rule conversion pipeline.

---

## Idea 17 — Generation Deficit vs Selection Deficit

### Problem first

Modern papers alternate between improving candidate generation and improving selection, but reported end-to-end gains obscure which bottleneck actually dominates for a model/benchmark.

### Decomposition

For each task:

```text
Pass@K = 0 -> generation deficit
Pass@K = 1 and selector wrong -> selection deficit
Pass@1 correct -> solved
```

Further divide generation deficit by semantic coverage from Idea 1.

### Cheap falsification

Run one modern generator with N=8/16 on BIRD, Spider and one enterprise split. Compute the proportion of total error attributable to each deficit.

Stop if the ratio is stable and already well established by R3-SQL/DPC analyses.

### Potential value

A cross-model scaling result may show that stronger models shift the bottleneck from generation to selection—or the reverse—changing where research effort should go.

---

## Idea 18 — Evidence-Source Substitution Matrix for interactive SQL

### Problem first

A database assistant can learn the same fact from user clarification, documentation, schema metadata, or a DB query. Existing interaction metrics count turns but do not measure source substitutability.

### Artifact

For BIRD-INTERACT-Lite, construct a task x evidence-source matrix:

```text
                  USER  KB  META  DB
fiscal calendar     1    1    0   0
status meaning      1    1    0   1
current table       0    1    1   1
business intent     1    0    0   0
```

### Metric

`source redundancy`, `user indispensability`, and `minimum source set`.

### Cheap falsification

Manual audit 20 tasks first.

Stop if evidence is almost always single-source, making substitution uninteresting.

### Research payoff

A positive result turns interactive Text2SQL into a constrained evidence-acquisition problem with a benchmark-derived ground truth, rather than inventing another generic routing mechanism.

---

# 4. Ideas rejected immediately in this pass

The anchor-first process also rejects several tempting extensions before implementation:

- SDE-SQL + reinforcement learning for choosing probes: too method-first before measuring whether probe choice is a real bottleneck.
- DPC + third programming language: does not solve shared upstream semantic misunderstanding.
- BIRD-INTERACT + better router: interaction-source necessity must be measured first.
- DeepEye + more verifier agents: stagewise first-error and regression accounting should come before adding modules.
- Spider2-AIFunc + more schema retrieval: the benchmark itself reports heavy traditional scaffolds often do not transfer.
- semantic layer + memory + RAG: first determine which semantic facts causally create the gain.
- another ambiguity benchmark: AmbiQT, PRACTIQ, CLARITY, AMBROSIA and Arcwise-Plat already cover substantial ambiguity space.
- generic candidate diversity: CHASE-SQL, XiYan-SQL and R3-SQL occupy this unless diversity is defined at a new semantic level.

---

# 5. Priority ranking after collision screening

| Rank | Direction | Core unanswered question | Real-data support | Cheap falsification | Novelty risk |
|---|---|---|---:|---:|---:|
| 1 | SemCov-SQL | Is candidate failure mainly missing semantic alternatives rather than weak selection? | 5/5 | 5/5 | Low-Med |
| 2 | AI-or-Not-SQL | Can agents decide whether AI semantics are needed before function generation? | 5/5 | 4/5 | Low |
| 3 | Irreducible Interaction Rate | How many BIRD-Interact user turns are actually necessary? | 5/5 | 4/5 | Low-Med |
| 4 | EvidenceSufficiency-SQL | What is the minimal evidence needed from active DB exploration? | 5/5 | 5/5 | Med |
| 5 | Semantic Annotation ROI | Which business facts create most semantic-layer value? | 4/5 | 5/5 | Low-Med |
| 6 | Shared-Blind-Spot SQL | Do cross-paradigm verifiers fail on shared semantic misunderstanding? | 5/5 | 5/5 | Med |
| 7 | Semantic First-Error Attribution | Where does the first wrong semantic commitment enter modern pipelines? | 5/5 | 4/5 | Med |
| 8 | Evidence Saturation Reversal | Can adding valid DB observations hurt final SQL? | 5/5 | 5/5 | Med |
| 9 | Semantic-Layer Generalization Audit | Is semantic-layer gain reusable knowledge or workload leakage? | 4/5 | 4/5 | Low |
| 10 | Evidence-Source Substitution Matrix | Which information sources are interchangeable in interactive SQL? | 5/5 | 4/5 | Med |
| 11 | Error Stickiness | Which early semantic mistakes survive downstream correction? | 5/5 | 4/5 | Med |
| 12 | Memory-as-Semantic-Layer | Are memory gains explained by declarative database semantics? | 4/5 | 4/5 | Med-High |

---

# 6. Best six low-cost pre-experiments

These are deliberately *not* full implementations.

## P0-1 — Semantic candidate coverage

- 30 BIRD questions;
- 8 candidates/question;
- manually or programmatically extract join/measure/grain/filter/time commitments;
- compare semantic coverage with oracle Pass@8.

Kill if semantic coverage is not more informative than AST/token diversity.

## P0-2 — DPC shared blind spots

- 30 DPC residual failures;
- label whether SQL/Python share the same wrong semantic interpretation.

Kill if shared-semantic failures are rare.

## P0-3 — SDE evidence deletion

- 30 successful SDE-SQL exploration trajectories;
- leave-one-probe-out and greedy evidence deletion;
- estimate redundant-probe fraction.

Kill if evidence is mostly non-redundant or replay is unreliable.

## P0-4 — BIRD-Interact information-source audit

- 20 Lite tasks with user turns;
- check whether each user-provided fact is available in DB/KB/metadata.

Kill if >90% is genuinely user-only.

## P0-5 — AIFunc operator-necessity pilot

- 20 ordinary Spider2 tasks + 20 AIFunc tasks;
- human-validated instructions without explicit platform function names;
- ask strong models only to classify whether deterministic SQL is sufficient.

Kill if strong models exceed ~95% balanced accuracy.

## P0-6 — semantic-layer fact ablation

- reproduce 20-50 examples from the open semantic-layer benchmark;
- atomize the semantic document;
- run small fact-subset / leave-one-fact-out ablations.

Kill if no small subset or stable fact-importance structure exists.

---

# 7. What would be a strong paper narrative?

The strongest candidates in this pass share one pattern:

> a successful 2026 method solves one layer of the problem, but its remaining failure is caused by an upstream scientific object that the benchmark/method does not explicitly represent.

Examples:

```text
DPC solves selection
    -> but candidate pools may lack semantic alternatives
    -> SemCov-SQL

SDE solves static context
    -> but active evidence may be mostly redundant/harmful
    -> EvidenceSufficiency-SQL

BIRD-INTERACT adds interaction
    -> but interaction may be substitutable by machine evidence
    -> Irreducible Interaction Rate

Spider2-AIFunc tests AI SQL generation
    -> but tells the model which AI semantics to use
    -> AI-or-Not-SQL

Semantic-layer work proves explicit semantics help
    -> but does not tell organizations what to document first
    -> Semantic Annotation ROI
```

This is the intended research style for subsequent passes.

---

# 8. References used in this pass

- SDE-SQL, ACL 2026: https://aclanthology.org/2026.acl-long.116/
- DPC, ACL 2026: https://aclanthology.org/2026.acl-long.313/
- BIRD-INTERACT, ICLR 2026 Oral: https://proceedings.iclr.cc/paper_files/paper/2026/hash/496b549556509bbb9770bf9d335c5800-Abstract-Conference.html
- DeepEye-SQL, SIGMOD 2026: https://arxiv.org/abs/2510.17586
- Spider 2.0-AIFunc: https://arxiv.org/abs/2607.06229
- R3-SQL, ACL Findings 2026: https://aclanthology.org/2026.findings-acl.2146/
- CHASE-SQL, ICLR 2025: https://proceedings.iclr.cc/paper_files/paper/2025/hash/974ff7b5bf08dbf9400b5d599a39c77f-Abstract-Conference.html
- MIRA: https://arxiv.org/abs/2608.06950
- GATE / Bootstrapping Semantic Layer from Execution: https://arxiv.org/abs/2606.05634
- Semantic-layer paired benchmark: https://arxiv.org/abs/2604.25149
- Semantic-layer benchmark code: https://github.com/cubedevinc/semantic-layer-benchmark/
- Benchmark annotation errors, VLDB 2026: https://github.com/uiuc-kang-lab/text_to_sql_benchmarks
- Context Length Alone Hurts LLM Performance, EMNLP Findings 2025: https://aclanthology.org/2025.findings-emnlp.1264/
- Spider2-AIFunc code: https://github.com/Leolty/Spider2-AIFunc
