# MetaSQL / Database-Harness Routing — Deep Audit (2026-08-24)

> Status: fresh research spike; no implementation changes.
> Goal: determine whether task-conditioned selection among materially different Text2SQL agent harnesses is still novel after 2025–2026 work on model routing, paradigm routing, mixture-of-agents, and adaptive computation.

## 1. Executive conclusion

Generic “routing” is heavily occupied. The direction survives only if the routing action is defined at the level of **database evidence and verification capabilities**, and only if a cross-regime oracle study first proves that different harnesses truly dominate on different SQL tasks.

A defensible thesis is:

> **Text2SQL should be evaluated and optimized as conditional selection among database reasoning harnesses with different observation/verification affordances, not as selection of a single globally best pipeline.**

The first contribution should be an **oracle-gap / architecture-reversal study**, not a learned router.

---

# 2. Generic routing is prior art

## 2.1 Select-then-Solve

Select-then-Solve (2026) compares Direct, CoT, ReAct, Plan-Execute, Reflection, and ReCode across four frontier models and ten benchmarks. No single reasoning paradigm dominates. Oracle per-task selection exceeds the best fixed paradigm by 17.1 points on average; a lightweight embedding router recovers part of the oracle gap.

Reference: https://arxiv.org/abs/2604.06753

Therefore:

> “different tasks need different agent paradigms”

is not novel.

## 2.2 RouteMoA

RouteMoA (ACL 2026) dynamically screens and ranks candidate models/agents, balancing quality, cost, and latency and reducing inference cost substantially relative to dense MoA.

Reference: https://aclanthology.org/2026.acl-long.558/

## 2.3 MoMA and generalized routing

MoMA (2025) explicitly integrates model routing and agent selection, using capability profiling and context-aware state-machine selection.

Reference: https://arxiv.org/abs/2509.07571

## 2.4 Session-aware agentic routing

2026 serving work also shows that agent routing must respect tool-loop continuity, non-portable backend state, cache economics, and session boundaries.

Implication: a Text2SQL paper cannot claim novelty for learned routing, cost-aware routing, or session-aware routing alone.

---

# 3. Why Text2SQL still gives an unusually strong routing hypothesis

Recent SQL/data-agent work provides concrete evidence that harness complexity is regime-dependent:

- Spider 2.0 exposes large-schema enterprise workflows where exploration/documentation/code context matter.
- BIRD-INTERACT makes interaction and CRUD central.
- BIRD-CRITIC/SWE-SQL is SQL debugging rather than forward generation.
- Effi-SQL separates final-query execution efficiency from correctness.
- Spider 2.0-AIFunc introduces AI-native operators and reports that traditional elaborate Text2SQL scaffolds do not transfer cleanly; minimal setups can be competitive or stronger.
- SDE-SQL/APEX-SQL show exploration helps in conventional/enterprise settings.
- DPC shows expensive cross-paradigm candidate verification can improve selection.

This suggests a sharper empirical claim:

> **The value of database-agent machinery is conditional on the SQL regime and on early observable uncertainty; extra modules can be negative-value computation.**

That is stronger than saying “ReAct sometimes beats CoT.”

---

# 4. Define a harness by affordances, not agent names

Avoid comparing arbitrary branded pipelines. Define a harness as a set of capabilities.

Example dimensions:

```text
schema access:
  none / retrieved / interactive

data access:
  none / samples / arbitrary read-only probes

candidate generation:
  1 / N

verification:
  parse / execute / tests / distinguishing DB / independent program

memory:
  none / examples / verified corrections

interaction:
  none / ask user

write capability:
  none / transactional / branchable

AI-native operators:
  unavailable / available
```

Concrete portfolio for an initial study:

```text
H0 Direct
H1 Retrieval-only
H2 SDE-like explorer
H3 APEX-like hypothesis explorer
H4 N-candidate + self-consistency
H5 DPC-like distinguishing verifier
H6 Memory-augmented correction
H7 Interactive ask/inspect/execute
```

The same underlying model should be used wherever possible so architecture effects are not model effects.

---

# 5. First paper should ask: is there an oracle harness gap?

For each task `x`, run all feasible harnesses under controlled accounting.

Define utility, for example:

```text
U(h,x) = success(h,x)
         - λ1 * LLM_cost
         - λ2 * DB_cost
         - λ3 * latency
         - λ4 * human_turns
```

Then:

```text
best_fixed = max_h mean_x U(h,x)

oracle = mean_x max_h U(h,x)

oracle_gap = oracle - best_fixed
```

If the oracle gap is small, no routing method can create a meaningful scientific result.

## Critical extra measurement: architecture reversal

For every pair `(h_i, h_j)`, measure task strata where:

```text
h_i >> h_j
```

and where the ordering reverses.

Potential strata:

- schema size;
- join count;
- documentation dependence;
- execution feedback need;
- ambiguity;
- AI-function count;
- interactive vs single-turn;
- mutation vs read-only;
- debugging vs generation.

A strong paper should reveal interpretable reversals, not just train a black-box router.

---

# 6. Cost equalization is the hardest methodology issue

A heavy harness usually receives more model calls and more observations. If it wins, reviewers can say the method is just test-time scaling.

Use at least two regimes:

## Natural-cost evaluation

Each harness runs as designed. Report total cost.

## Equal-budget evaluation

Provide an identical upper bound on:

- LLM input/output tokens or dollar cost;
- number of DB executions;
- maximum wall time;
- user turns.

This tests whether architecture matters beyond raw compute.

Also report a Pareto frontier rather than one scalar score.

---

# 7. What features may predict the best harness?

Only after establishing an oracle gap should routing be studied.

Feature families:

## Static task features

- schema table/column count;
- query length;
- detected entity count;
- ambiguity cues;
- dialect;
- AI-function availability;
- read/write task classification.

## Cheap schema features

- candidate-table entropy;
- FK graph density;
- lexical schema match margin;
- number of plausible join paths.

## Early-trajectory features

- first-pass SQL parse status;
- execution empty/non-empty/error;
- candidate disagreement;
- verifier warning count;
- schema-probe novelty;
- confidence/calibration signals.

## Cost state

- remaining token budget;
- DB-query budget;
- latency SLO;
- human-interruption budget.

A dynamic router could start with H0/H1 and escalate after early evidence, but this should be a later paper.

---

# 8. Strongest new formulation: Capability Escalation rather than one-shot routing

One-shot selection before seeing any DB evidence may be too weak.

A more database-native formulation is:

```text
start minimal
   ↓
observe cheap failure/uncertainty signals
   ↓
activate exactly one new capability
   ↓
re-evaluate
```

Actions could be:

```text
+ schema retrieval
+ DB probe access
+ second candidate
+ expensive verifier
+ memory retrieval
+ user clarification
+ stronger model
```

This is adaptive computation over **information and verification affordances**.

Potential objective:

```text
capability* = argmax_a expected marginal task utility(a) - cost(a)
```

This overlaps general adaptive computation, but the SQL-specific action space is concrete and measurable.

---

# 9. Killer experiment 0 — architecture reversals

Use a modest cross-regime set before building a router:

- BIRD subset;
- Spider2-Snow subset;
- Spider2-AIFunc subset;
- Mini/BIRD-Interact subset if environment cost allows;
- SQL debugging subset.

Run 4–6 deliberately different harnesses.

The experiment succeeds if:

1. no harness dominates every regime;
2. oracle selection gives substantial gain over best fixed;
3. task features predict at least some reversals;
4. the conclusion survives cost equalization.

### Suggested go/no-go

Proceed only if:

- oracle improvement ≥5 absolute success points **or** ≥15% cost reduction at matched success;
- at least two robust architecture reversals appear across regimes.

Otherwise the problem is not large enough.

---

# 10. Learned router only after the oracle study

Baselines:

1. best fixed harness;
2. rule-based regime router;
3. LLM self-router;
4. embedding classifier similar in spirit to Select-then-Solve;
5. cost-sensitive supervised router;
6. dynamic escalation controller;
7. oracle.

Primary result should be fraction of oracle gap recovered at a given cost, not only raw accuracy.

---

# 11. Potential benchmark: No-One-SQL-Agent

A useful benchmark would not crown one SQL method. It would measure **harness generality**.

For harness `h`, define:

```text
generality_regret(h) = average over regimes of
  [best_in_regime - h]
```

Also measure worst-regime regret.

This can expose methods that lead one leaderboard but fail badly elsewhere.

Add task-level metadata for:

- permitted observations;
- interaction availability;
- mutation risk;
- evaluation cost;
- AI-native operator use.

This could be valuable even without a learned router.

---

# 12. New derived ideas

## M1. Negative-Value Module Benchmark

Quantify how often adding one agent module decreases success. Modules: RAG, reflection, exploration, memory, verifier, multi-candidate sampling.

## M2. Harness Shapley Value

Estimate each module’s marginal contribution across task distributions, including negative interactions.

## M3. Failure-Opportunity Accounting

Every extra module can introduce an error. Measure module-local corruption events such as retrieval exclusion, repair regression, verifier false rejection, and memory poisoning.

## M4. Complexity Break-Even Curves

Find where the benefit of a module crosses zero as schema size, ambiguity, or query complexity increases.

## M5. Escalation Calibration

Evaluate whether an agent’s predicted “need for more machinery” is calibrated against actual counterfactual benefit.

## M6. Early-Exit Text2SQL

Stop once further modules have negative expected marginal value. Compare with fixed full pipelines.

## M7. Harness Distillation

Use expensive oracle traces to learn a small deterministic decision tree that selects capabilities, prioritizing interpretability over router-model complexity.

## M8. Architecture Drift

As base LLMs improve, formerly useful modules may become harmful. Re-evaluate harness value across model generations and measure when scaffolds should be retired.

## M9. Tool-Capability Pricing

Assign each capability a real cost and optimize a per-query “shopping cart” of reasoning resources.

## M10. Cross-Dialect Harness Transfer

Test whether routing rules learned on SQLite/Postgres transfer to Snowflake/BigQuery, or whether database platform changes the optimal scaffold.

---

# 13. Falsification conditions

Kill or downgrade MetaSQL if:

1. one harness dominates nearly all fresh regimes;
2. architecture reversals vanish under equal compute;
3. the oracle gap is small;
4. simple static regime labels achieve nearly oracle performance, making a research-grade router unnecessary;
5. routing features mostly identify benchmark identity rather than task properties;
6. routing does not transfer across base model changes.

---

# 14. Revised judgment

Generic paradigm/model/agent routing is clearly occupied. The research opportunity is narrower:

> **measure and exploit conditional value of database-specific evidence and verification capabilities across heterogeneous SQL regimes.**

The strongest first paper may actually be an empirical/benchmark paper rather than a router paper:

> **No One SQL Agent: When Text2SQL Scaffolds Help, Hurt, and Reverse Across Modern Data-Agent Regimes**

If the oracle gap is large, a second paper can introduce adaptive capability escalation.

Revised score:

- Novelty: 4/5
- Feasibility: 5/5
- Scientific clarity: 5/5
- Main risk: generic routing prior art
- Best first action: low-cost oracle-gap experiment, not router engineering
