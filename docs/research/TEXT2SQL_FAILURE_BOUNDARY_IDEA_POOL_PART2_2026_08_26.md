# Text2SQL Failure-Boundary Idea Pool — Part II (2026-08-26)

> Status: literature-first research spike; no implementation changes.
> Goal: continue generating ideas from failure boundaries of strong 2026 papers rather than from method names.
> Rule: every direction must start from an empirical claim, identify one held-fixed assumption, define a low-cost falsification experiment, and specify a kill condition before any system building.

---

## 1. Why this pass is stricter than an ordinary idea brainstorm

The current Text2SQL/data-agent frontier is already crowded with database probing, parallel exploration, clause-level rewards, tree-structured correction, candidate verification, semantic layers, agent-friendly schema design, AI-native SQL, and long-horizon agent workflows. The risk is therefore no longer a lack of mechanisms; it is mistaking a mechanism transfer for a new scientific problem.

This pass uses the following chain:

```text
strong anchor result
    -> what variable did the paper intentionally hold fixed?
    -> does the claim fail when that variable changes?
    -> can the failure be measured on existing real benchmark artifacts?
    -> can a 20–50 case pilot kill the idea cheaply?
    -> only if the phenomenon survives do we design a method
```

Anchor papers / systems used heavily in this pass:

- Parekh et al., **PExA: Parallel Exploration Agent for Complex Text-to-SQL**, ACL 2026 Short. https://aclanthology.org/2026.acl-short.48/
- Lee et al., **EXPO-SQL: Execution-based Clause-level Policy Optimization for Text-to-SQL**, ACL Findings 2026. https://aclanthology.org/2026.findings-acl.1107/
- Li et al., **DPC: Training-Free Text-to-SQL Candidate Selection via Dual-Paradigm Consistency**, ACL 2026. https://aclanthology.org/2026.acl-long.313/
- Huang et al., **ACTS-SQL**, 2026. https://arxiv.org/abs/2608.15145
- Kim et al., **A Semantic-Layer-Mediated Agent for Natural Language to SQL over Heterogeneous Enterprise Databases**, 2026. https://arxiv.org/abs/2606.31041
- Rumiantsau & Fokeev, **Semantic Layers for Reliable LLM-Powered Data Analytics**, 2026. https://arxiv.org/abs/2604.25149
- Kanchinadam et al., **Same Data, Different Schemas**, 2026. https://arxiv.org/abs/2605.25838
- Su et al., **Disentangling Structure and Semantics**, 2026. https://arxiv.org/abs/2608.20356
- Zhang et al., **The Case for Text-to-SQL Friendly Logical Database Design**, 2026. https://arxiv.org/abs/2606.03145
- Liu et al., **Spider 2.0-AIFunc**, 2026. https://arxiv.org/abs/2607.06229
- Lin et al., **SEMA-SQL**, 2026. https://arxiv.org/abs/2604.23477
- Mang et al., **Horrila**, 2026. https://arxiv.org/abs/2604.09944
- Ha et al., **CADENZA**, 2026. https://arxiv.org/abs/2606.29151

The important observation is that several of these papers themselves expose promising cracks: PExA assumes test cases can be made independent; EXPO-SQL explicitly admits that unique clause-error identification from execution is theoretically intractable; DPC constructs synthetic distinguishing worlds that are schema-valid but not necessarily business-valid; agent-friendly logical design reports real regressions when aggressive partitioning prunes required tables or retrieves a wrong view; semantic-layer systems acknowledge quality/overfitting concerns; AI-native SQL systems optimize cost/quality under stochastic operators but still inherit classical rewrite assumptions in parts of the stack.

---

# 2. Idea 1 — InteractionFaultSQL: when every component is locally correct but the whole query is wrong

## Anchor result

PExA frames Text2SQL as semantic test coverage. It creates simpler, self-contained SQL test cases that are intentionally independent and executes them in parallel before integrating their evidence into the final SQL. EXPO-SQL decomposes SQL by logical execution order and provides clause-level rewards from incremental execution.

These are strong results, but both rely on a form of **locality**: useful evidence or blame can be attached to a test case or clause in isolation.

## Failure boundary

Many SQL errors are not local. They emerge only when two individually reasonable decisions interact.

Examples:

```text
LEFT JOIN + WHERE predicate on right table
    -> silently becomes INNER-like filtering

many-to-many JOIN + SUM
    -> fan-out inflation

DISTINCT + aggregate
    -> hides duplicate-generation instead of fixing grain

window PARTITION BY + post-window filter
    -> wrong ranking population

temporal JOIN + GROUP BY
    -> multiple validity intervals counted together

filter + LIMIT/order
    -> correct components, wrong order of operations
```

A unit test for the join can pass. A unit test for the aggregate can pass. Their composition can still be wrong.

EXPO-SQL explicitly notes that unique clause identification from finite execution is theoretically incomplete and handles nested subqueries, windows, and CTEs at enclosing-clause granularity. This suggests a concrete empirical question rather than an immediate new method:

> **How much residual Text2SQL error is caused by semantic interactions that cannot be assigned to one locally incorrect clause/test?**

## P0 falsification experiment

Take 30–50 residual failures from PExA / EXPO-SQL / strong direct baselines where gold SQL is available.

For each failure:

1. identify semantic requirements in the question;
2. test the corresponding SQL components individually;
3. test pairwise composition;
4. mark the smallest interaction order at which the output diverges from gold.

Label:

```text
LOCAL_1
PAIRWISE_2
HIGHER_ORDER_3+
GLOBAL_MISINTERPRETATION
```

Primary endpoint:

```text
P(interaction order >= 2 | executable wrong query)
```

## Kill condition

If fewer than ~10% of audited semantic failures require pairwise-or-higher interaction to explain, stop. Existing clause/test decomposition is probably enough.

## If the phenomenon survives

The paper should initially be a **failure-boundary / evaluation paper**, not “add pairwise agents.”

A stronger second-stage contribution could be an interaction-coverage metric or combinatorial semantic test suite, but only after proving that nonlocality is a material failure source.

## Why this is differentiated

This is not fault localization. It asks whether the assumption that a fault *has a localizable single component* is itself wrong.

Potential venue flavor: ACL/EMNLP if framed as agent reasoning; SIGMOD/VLDB if formalized around SQL operator interactions.

---

# 3. Idea 2 — FeasibleWorldSQL: does synthetic verification distinguish candidates using impossible business worlds?

## Anchor result

DPC constructs a Minimal Distinguishing Database (MDD): a small synthetic database where two candidate SQL queries return different outputs, then solves the same NL request with Python/Pandas to select the candidate.

The TESTER prompt enforces schema names, types, and foreign-key relationships. The paper's case study creates a “ghost transaction” that exists in one table but lacks a corresponding monthly record so that competing joins diverge.

## Failure boundary

A database instance can satisfy declared SQL schema constraints and still be **impossible under the business process**.

Examples of latent constraints that are often not present as PK/FK/UNIQUE declarations:

```text
captured payment must belong to a settled order
one active subscription per account and product
shipment timestamp >= order approval timestamp
recognized revenue only after delivery
closed support case must have a resolution event
financial period rows must exist for every booked transaction
```

If candidate A and candidate B differ only on worlds that the real organization can never produce, then “distinguishability” is not automatically evidence that one candidate better represents the user intent.

The deeper question is:

> **Should candidate SQLs be considered semantically different if they diverge only outside the feasible data-generating process?**

## P0 falsification experiment

Use 30–50 DPC artifacts where an MDD changed or strongly supported the final selection.

For each MDD:

1. collect declared schema constraints;
2. collect benchmark evidence / column descriptions / value descriptions;
3. infer only high-confidence empirical invariants from the original database;
4. audit whether the MDD violates any of these invariants;
5. regenerate or filter distinguishing worlds under the invariant set;
6. check whether the selected candidate changes.

Primary metrics:

```text
MDD latent-constraint violation rate
selection flip rate after feasibility filtering
fraction of candidate distinctions that disappear in feasible worlds
```

## Kill condition

If <5% of MDD decisions involve a plausible latent-constraint violation or feasibility filtering almost never changes the selection evidence, drop the direction.

## If the phenomenon survives

The scientific object is **feasible-world semantic equivalence**, not “add constraints to DPC.”

Two SQLs would be equivalent relative to a feasible-world theory F when:

```text
for every legal database D satisfying F:
    Q1(D) = Q2(D)
```

This connects candidate verification to database constraints and domain semantics while remaining anchored to a concrete observed failure mode.

Potential venue: SIGMOD/VLDB/PODS-adjacent database-agent paper.

---

# 4. Idea 3 — UnidentifiableSQL: a lower bound on what database exploration can ever resolve

## Anchor result

SDE-SQL, PV-SQL, APEX-style systems, PExA, and DPC all increase external evidence. Their shared premise is that executing the right database observations can resolve uncertainty that static schema prompting cannot.

## Failure boundary

Sometimes the current database instance simply contains no evidence that distinguishes two different semantic interpretations.

Examples:

```text
invoice.total == captured_payment for every row today

billing_region == sales_region for every current customer

two candidate join paths happen to cover exactly the same entity pairs

status_history contains only one row per entity in this snapshot

all refund rows are zero in the current period
```

The agent can inspect every row and still not learn which interpretation the organization intends.

This is not an exploration-policy failure. It is an **identifiability failure**.

## Research question

> For what fraction of realistic Text2SQL ambiguities is the intended semantics identifiable from the current database instance at all?

## P0 falsification experiment

Start with candidate pairs from BIRD/Spider where both SQLs execute to the same answer on the original database but encode different semantics.

For each pair:

1. classify the semantic difference;
2. allow unrestricted read-only probing of the original database;
3. independently construct one or more legal alternate database instances using published test-suite / distinguishing-database tools;
4. determine whether the candidate pair diverges on a legal alternate instance.

The interesting set is:

```text
same answer on current D
+ semantically distinct programs
+ diverge on another legal D'
```

That set represents semantics that **cannot be identified from the snapshot alone**.

## Primary metric

```text
Snapshot Non-Identifiability Rate
```

conditioned on real candidate disagreement classes.

## Kill condition

If nearly all semantically distinct candidate pairs can be resolved by ordinary probing of the actual database, stop.

## Why this matters

It creates a principled boundary between:

```text
more DB exploration will help
vs
only external semantics / user input / governance can help
```

That is a more fundamental result than designing another probe-selection policy.

Potential venue: ACL/ICLR for agent epistemics; SIGMOD/VLDB if formalized through database-instance distinguishability.

---

# 5. Idea 4 — Repair Basin Geometry: when is SQL correction the wrong task formulation?

## Anchor result

ACTS-SQL shows that plan-guided tree correction, multiple strategies, execution verification, and backtracking materially improve SQL correction on BIRD-Critic and in a production Text-to-TLS setting.

The natural next question is not “add another correction strategy.” It is:

> **Which incorrect SQL queries are actually repairable from their current state?**

## Failure boundary

Correction assumes the candidate lies in a useful neighborhood of the target.

But two wrong queries with similar token/AST edit distance can differ radically:

```text
Case A:
correct tables + correct grain + one wrong date boundary

Case B:
wrong business entity + wrong source-of-truth table + wrong result grain
```

Both may need only a handful of textual edits, but B is conceptually a regeneration problem.

## P0 falsification experiment

Use real BIRD-Critic / SWE-SQL faulty SQLs and, where possible, real model errors rather than synthetic mutations.

Annotate each input along independent dimensions:

```text
intent alignment
schema grounding alignment
grain alignment
join skeleton alignment
aggregation alignment
number of coupled semantic errors
```

Run 2–3 strong correction methods under equal budget.

Fit a simple model predicting correction success from these dimensions.

## Hypothesis

There exists a sharp **repairability frontier**: once a small number of upstream semantic commitments are wrong simultaneously, additional local correction search has sharply diminishing return.

## Kill condition

If correction probability varies smoothly with ordinary query difficulty and no stable boundary emerges across models/methods, do not pursue.

## If it survives

The first paper can be “Correction is not always the right problem” plus a repairability benchmark/geometry.

Only later should one consider a repair-vs-regenerate policy; doing that first would collapse back into generic routing.

---

# 6. Idea 5 — SemanticIR-Coverage: the hidden expressiveness ceiling of semantic-layer Text2SQL

## Anchor result

The semantic-layer-mediated Spider2 system reasons over a compact Semantic Model Query (SMQ) and deterministically compiles the SMQ to backend-specific SQL, obtaining very high execution accuracy. The paper explicitly discusses semantic-layer quality and overfitting trade-offs.

## Failure boundary

A semantic intermediate representation improves reliability partly because it restricts the program space.

But every restricted IR has an expressiveness boundary.

Questions that require:

```text
rare correlated subqueries
unusual window semantics
noncanonical temporal logic
ad-hoc derived measures
nonstandard set operations
one-off cross-domain joins
backend-specific constructs
```

may be answerable in SQL while not naturally representable in the semantic IR.

A benchmark containing mostly IR-expressible tasks can make semantic mediation look universally superior while hiding a **coverage ceiling**.

## P0 falsification experiment

Take a representative sample of 50–100 Spider2-Snow tasks, including tasks the semantic-layer system fails.

For each task, classify:

```text
DIRECTLY_EXPRESSIBLE_IN_IR
EXPRESSIBLE_WITH_COMPOSITION
REQUIRES_ESCAPE_HATCH
NOT_EXPRESSIBLE
```

Then separate system failure into:

```text
NL -> semantic IR error
semantic IR coverage failure
compiler/backend error
```

## Kill condition

If >95% of tasks are naturally representable and IR coverage explains almost none of the failures, stop.

## If it survives

The scientific question is not “make the IR bigger.” It is:

> How should a reliable data agent expose a constrained semantic language while preserving access to the long tail of valid SQL expressivity?

This is analogous to language/runtime escape-hatch design, but the contribution would be grounded in measured enterprise query coverage.

Potential venue: SIGMOD/VLDB.

---

# 7. Idea 6 — What Does a Semantic Layer Actually Buy? Causal decomposition of the reported gain

## Anchor result

A paired 2026 benchmark reports +17 to +23 percentage-point gains when frontier models receive a small hand-authored document describing measures, conventions, and disambiguation rules. The semantic-layer-mediated Spider2 system similarly reports large gains relative to schema-only approaches.

## Failure boundary

“Semantic layer” bundles multiple interventions:

```text
business definitions
source-of-truth table hints
join path hints
metric formulas
schema compression
naming clarification
result-grain constraints
removal of irrelevant alternatives
```

If 80% of the gain comes from one simple factor—say oracle table narrowing—then the scientific story is different from “business semantics are the dominant bottleneck.”

## P0 falsification experiment

Use the paired semantic-layer dataset or a reproducible subset.

Factor the semantic context into disjoint treatments:

```text
T0 schema only
T1 only metric/business definitions
T2 only join/source-of-truth rules
T3 only naming/aliases
T4 only table narrowing with no added semantics
T5 full semantic layer
T6 token-matched irrelevant control text
```

Keep model, prompt budget, and benchmark fixed.

## Primary endpoint

ANOVA-style or matched-task decomposition of execution-accuracy gain.

## Kill condition

If the full gain is diffuse and no stable mechanism reproduces across models/domains, do not build a new method around any single factor.

## Why this is valuable

This is a mechanism-discovery paper, not a semantic-layer product paper.

It directly answers what practitioners should actually invest in and prevents future work from attributing a bundled intervention to the wrong cause.

---

# 8. Idea 7 — Long-Tail Schema Regression: workload-aware schema optimization may erase rare intents

## Anchor result

The 2026 paper on Text2SQL-friendly logical database design introduces schema abstraction, workload-aware partitioning, and descriptive renaming. It reports net improvements but also an important failure analysis: 42 losses stem largely from aggressive partitioning that prunes a required table or retrieval of an incorrect view.

The method mines frequent table sets and recurring join structures from historical workloads.

## Failure boundary

Workload-aware optimization naturally favors frequent intents.

That raises a classical systems question in a new setting:

> **Does optimizing the schema representation for common historical questions systematically harm rare or novel analytical intents?**

This is more specific than generic distribution shift.

## P0 falsification experiment

Using the authors' workload-history setup:

1. compute support/frequency of each test query's required table-set / join motif in historical workload H;
2. bin tasks by support quantile;
3. compare baseline vs +A/+P/+R accuracy and regression rate in each bin;
4. repeat with held-out domains if available.

Primary metric:

```text
RegressionRate(method | workload-support decile)
```

## Hypothesis

Average gains mask a tail-risk curve: the less historical support a valid intent has, the more likely an optimized model-facing schema is to remove or distort the information needed to answer it.

## Kill condition

If regressions are not concentrated in low-support / novel-query regions, stop.

## If it survives

This establishes a new objective for agent-facing database design:

```text
maximize average generation accuracy
subject to a bound on long-tail semantic coverage loss
```

The first contribution is the measured trade-off, not a new router.

Potential venue: SIGMOD/VLDB.

---

# 9. Idea 8 — Cognitive–Physical Schema Tradeoff: is the easiest schema for an LLM expensive for the database?

## Anchor results

Same-Data-Different-Schemas and the 2026 structure-vs-semantics factorial study show that equivalent schema representations materially change Text2SQL accuracy. Text2SQL-friendly logical design proposes views, partitioning, and renaming as a new database design objective.

The latest paper explicitly points toward future cost-aware view materialization, but current evaluations optimize the model-facing generation objective separately from ordinary database execution/maintenance costs.

## Failure boundary

A representation that makes query generation easier can increase:

```text
view maintenance cost
storage
query-plan complexity
refresh latency
write amplification
schema migration burden
security/governance surface
```

Conversely, a highly normalized physical model may be efficient/maintainable but cognitively hostile to an LLM.

## P0 falsification experiment

Use published equivalent-schema variants and +A/+P/+R transformations.

For each representation measure:

```text
Text2SQL execution accuracy
prompt/context size
SQL complexity
DB optimizer estimated cost
actual runtime on a controlled database
view/materialization maintenance cost when applicable
```

## Hypothesis

There is a nontrivial Pareto frontier between **cognitive accessibility for the generator** and **physical/operational cost for the DBMS**.

## Kill condition

If model-friendly transformations consistently improve or leave classical DB costs unchanged, the interesting trade-off is weak; stop and treat the transformations as a straightforward engineering win.

## If it survives

The research problem becomes joint database design for two consumers:

```text
LLM query generator
DB execution engine
```

This is stronger than another Text2SQL method because it creates a real multi-objective database design problem.

---

# 10. Idea 9 — RewriteSoundSQL: which relational rewrite laws remain sound with AI operators?

## Anchor results

Spider2-AIFunc introduces SQL containing AI functions. SEMA-SQL, Horrila, CADENZA, and related semantic-query systems combine relational and LLM-powered operators and apply optimization/rewrite rules to reduce cost.

Horrila, for example, studies equivalence-preserving placement rewrites and evaluates output quality rather than assuming ordinary deterministic relational semantics is enough.

## Failure boundary

Classical relational rewrites assume operators are effectively pure and deterministic under the relevant semantics.

AI operators can be:

```text
nondeterministic
context-sensitive
prompt-sensitive
backend/version-dependent
non-extensional
asymmetric in false-positive / false-negative behavior
```

Then two SQL plans that are algebraically equivalent in ordinary relational logic can yield different answers after reordering an AI predicate.

Examples worth testing:

```text
semantic filter before vs after deduplication
semantic predicate before vs after join expansion
semantic ranking before vs after grouping
AI extraction once per unique value vs once per duplicated row
semantic join with candidate pruning before vs after relational filter
```

## P0 falsification experiment

Select 20–30 AI-SQL queries from Spider2-AIFunc / SemBench-like workloads.

Generate only rewrite pairs that are equivalent under deterministic relational assumptions.

Run each pair repeatedly under fixed backend/model settings and measure:

```text
output divergence
F1 divergence against labels where available
cost/latency difference
whether divergence exceeds ordinary repeated-run noise
```

## Kill condition

If rewrite-induced semantic divergence is negligible relative to repeated execution noise, classical equivalence assumptions are adequate for this regime; stop.

## If it survives

The problem is **semantic rewrite soundness**, not another query optimizer.

A later method might attach statistical preconditions to rewrite rules, but the first result should establish exactly which algebraic laws fail and under what operator conditions.

Potential venue: SIGMOD/VLDB/PODS-style systems-semantics work.

---

# 11. Idea 10 — ErrorAmplificationSQL: small AI-operator mistakes can become large relational answer errors

## Anchor result

Spider2-AIFunc deliberately verifies tasks across repeated temporal windows to keep benchmark answers stable. Current semantic-query systems optimize quality, latency, and cost, often reporting operator-level F1 or query-level quality.

## Failure boundary

The same per-row AI error rate can have radically different final impact depending on where the operator sits in the relational plan.

Examples:

```text
5% classification error before COUNT
5% error before SUM with highly skewed amounts
5% false positives before many-to-many JOIN
one ranking swap before TOP-1
one extraction error feeding a GROUP BY key
```

This is an **error propagation topology** problem.

## P0 falsification experiment

For 20–30 AI-SQL tasks:

1. measure or approximate operator-level error/stability with repeated calls or available labels;
2. perturb only observed operator mistakes, not arbitrary database noise;
3. measure final answer sensitivity by downstream relational structure;
4. regress amplification on operator position, selectivity, skew, join fan-out, aggregation type, and top-k sensitivity.

Define:

```text
Amplification = final-answer error / local semantic-operator error
```

## Kill condition

If final error is roughly proportional to local operator error and query topology explains little additional variance, stop.

## If it survives

This creates a new optimizer/evaluator objective: not all semantic operator errors are equally dangerous.

The database should spend quality budget where local mistakes have the largest downstream influence.

---

# 12. Idea 11 — Benchmark Mechanism Fragility: do conclusions survive benchmark revisions?

## Anchor observation

Spider 2.0 maintains an explicit data-update log and has revised ambiguous examples and gold outputs. PExA states that much of its ablation/analysis was performed under an older evaluation setup while its headline updated result uses newer evaluation data; the paper cautions that exact metric values can differ after gold updates.

A separate 2026 audit reports pervasive annotation errors in major Text2SQL benchmarks and shows that corrections can alter system rankings.

## Failure boundary

A method paper usually treats its ablation conclusion as stable:

```text
component X gives +2.9
component Y gives +0.6
```

But if benchmark corrections change not only the overall score but **which mechanism appears useful**, then component claims are less reproducible than headline accuracy suggests.

## P0 falsification experiment

Select 2–3 open methods with released predictions/trajectories and benchmark versions.

Re-evaluate:

```text
full method
key ablations
error-category breakdowns
```

across two benchmark revisions / corrected labels where possible.

Primary endpoint:

```text
sign stability and rank stability of ablation effects
```

## Kill condition

If component effect directions are highly stable despite label revisions, drop this idea.

## Why keep it lower priority

This is scientifically useful but more meta-evaluation than a core Text2SQL mechanism. It belongs below the more task-intrinsic ideas above unless the instability is surprisingly severe.

---

# 13. Idea 12 — Integration Completeness: does “semantic coverage” measure the right thing?

## Anchor result

PExA explicitly states that its atomic test cases collectively cover the semantics of the original question, and the proposer synthesizes the final SQL from the distilled evidence. Its semantic verifier has a relatively small ablation impact compared with the parallel exploration components.

## Failure boundary

Coverage of requirements does not imply successful **integration** of those requirements.

A test suite may separately establish:

```text
correct customer population
correct revenue measure
correct quarter filter
correct refund treatment
```

while the final query integrates them at the wrong grain or in the wrong order.

This differs subtly from Idea 1:

- Idea 1 asks whether the error itself is nonlocal.
- This idea asks whether a coverage metric can be high even when the integration step is structurally incapable of proving the composition.

## P0 falsification experiment

On 30 PExA trajectories:

1. manually/automatically score whether the test suite contains evidence for every gold semantic requirement;
2. compare coverage score with final correctness;
3. inspect the set `high coverage + wrong final SQL`;
4. determine whether failures concentrate in integration/grain/order rather than missing evidence.

## Kill condition

If high semantic coverage almost always implies a correct final query, the current formulation is adequate; stop.

## If it survives

A future metric must distinguish:

```text
requirement coverage
interaction coverage
integration correctness
```

This could mature into a sharper complexity measure for long-form SQL tasks.

---

# 14. Current priority after this pass

The ideas below are ranked by a combination of novelty, ability to use existing real artifacts, cheap falsifiability, and clarity of contribution boundary.

| Rank | Direction | Core problem | P0 cost | Novelty risk | Why it is attractive |
|---|---|---|---|---|---|
| 1 | InteractionFaultSQL | local decomposition misses cross-clause semantics | low | low-medium | uses PExA/EXPO residuals; clean kill condition |
| 2 | FeasibleWorldSQL | synthetic distinguishing DB may rely on impossible worlds | low-medium | low | directly audits ACL'26 DPC artifacts |
| 3 | UnidentifiableSQL | current snapshot cannot contain enough evidence | medium | low | establishes a principled lower bound on exploration |
| 4 | SemanticIR-Coverage | semantic layers may have an expressiveness ceiling | low-medium | low-medium | highly relevant if semantic layers become standard |
| 5 | Long-Tail Schema Regression | workload optimization may sacrifice rare intents | low | low | anchor paper already reports regressions; no new system needed initially |
| 6 | RewriteSoundSQL | stochastic AI operators may invalidate rewrite assumptions | medium | low | deep DB semantics + fast-growing AI-SQL area |
| 7 | Repair Basin Geometry | some candidates should not be repaired at all | medium | medium | strong production anchor via ACTS-SQL |
| 8 | Semantic-Layer Gain Decomposition | bundled semantic context obscures causal mechanism | low | medium | practical and easy to falsify |
| 9 | Cognitive–Physical Schema Tradeoff | LLM-friendly logical design may impose DB cost | medium | medium | genuine DB co-design problem |
| 10 | ErrorAmplificationSQL | local AI error has topology-dependent query impact | medium-high | low-medium | strong systems angle, but execution cost is higher |
| 11 | Integration Completeness | coverage != successful composition | low | medium | close to InteractionFaultSQL; keep only if empirically distinct |
| 12 | Benchmark Mechanism Fragility | component claims may depend on benchmark version | low | medium-high | useful but more meta-science than core Text2SQL |

---

# 15. Recommended next P0 experiments before any large implementation

The next machine-side work should be small, diagnostic, and allowed to kill most of the pool.

## P0-A — Interaction faults

```text
30–50 executable wrong SQLs
-> identify local vs pairwise/higher-order fault
-> STOP if interaction-order >=2 <10%
```

## P0-B — DPC feasible-world audit

```text
30–50 MDD decisions
-> audit declared + high-confidence latent constraints
-> regenerate/filter infeasible worlds
-> STOP if selection flip/evidence loss <5%
```

## P0-C — Snapshot identifiability

```text
30 candidate pairs identical on real DB
-> prove semantic distinction on legal alternate DBs
-> test whether any real-data probe can distinguish
-> STOP if almost all are identifiable from current snapshot
```

## P0-D — Semantic IR coverage

```text
50–100 Spider2 tasks
-> classify IR representability
-> decompose failures
-> STOP if natural IR coverage >95% and coverage explains few errors
```

## P0-E — Long-tail schema regressions

```text
existing workload-aware schema-design benchmark
-> correlate regression with historical table-set/join-motif support
-> STOP if no tail concentration
```

## P0-F — AI rewrite soundness

```text
20–30 AI-SQL queries
-> relationally equivalent rewrite pairs
-> repeated execution
-> STOP if rewrite divergence ~= ordinary run-to-run noise
```

Only after these P0s should the project choose one or two ideas for actual method development.

---

# 16. Ideas deliberately rejected in this pass

The following temptations appeared during literature review but were rejected because they repeat already occupied spaces or previous failed patterns:

- another RL policy for choosing database probes;
- another clause-level reward scheme before proving clause locality is the right abstraction;
- another correction tree/search method after ACTS-SQL;
- another DPC variant with a third programming language;
- another semantic-layer agent with more RAG;
- generic “schema routing” after adaptive/schema-design work;
- generic provenance-based repair;
- generic memory invalidation/TMS;
- generic uncertainty calibration;
- generic multi-agent voting;
- synthetic drift benchmark without a real anchor phenomenon;
- “AI-SQL is stochastic” as a standalone contribution, because semantic-query systems already optimize under stochastic quality/cost.

The surviving ideas above all attack an assumption that a strong result currently relies on.

---

# 17. Core thesis emerging from the new process

The most promising research questions are no longer “what module should we add?” They look more like:

```text
What information is fundamentally identifiable from one database snapshot?

When is a SQL error inherently nonlocal?

When does a synthetic verification world cease to be semantically legal?

What legitimate query space is excluded by a semantic intermediate language?

Does optimizing a schema for common LLM workloads harm rare intents?

Which algebraic assumptions break once AI becomes a query operator?

When is an incorrect SQL outside the basin of meaningful correction?
```

These questions have three useful properties:

1. a negative result is scientifically interpretable;
2. the phenomenon can be tested before building a large agent;
3. if the phenomenon exists, the method contribution follows from the failure mechanism instead of being chosen in advance.

That is the research discipline this pool should continue to use.