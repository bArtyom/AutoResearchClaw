# ActiveProbeSQL — Deep Novelty Audit for Experimental Text2SQL (2026-08-24)

> Status: literature-first spike; no implementation changes.
> Goal: test whether explicit experimental-design control over database observations is a publishable step beyond SDE-SQL/APEX-SQL and beyond general LLM information-gathering work.

## 1. Executive conclusion

The naive proposal

> “let a Text2SQL agent issue exploratory SQL probes before writing the final query”

is already occupied.

The broader proposal

> “choose questions/queries by expected information gain”

is also occupied in general LLM-agent research.

The direction survives only as a database-specific decision problem:

> **Given an explicit distribution over competing relational/semantic hypotheses and a budgeted set of legal observations over the real database, can a Text2SQL agent choose the next database experiment to maximize expected downstream decision value, and does this reduce the number/cost of observations needed to reach a correct query?**

A better working name is:

> **BED-SQL: Bayesian Experimental Design for Database Grounding**

or

> **ActiveProbeSQL: Cost-Aware Discriminative Database Experiments for Text2SQL**.

---

# 2. Collision 1 — SDE-SQL already uses SQL probes

SDE-SQL (ACL 2026) explicitly performs Self-Driven Exploration by generating and executing SQL probes during inference. It uses exploration before generation and during refinement, including targeted probes for diagnosing empty or problematic results.

Reference: https://aclanthology.org/2026.acl-long.116/

The official description reports an 8.02% relative execution-accuracy improvement over its vanilla open-model baseline.

Therefore “SQL probes” are prior art.

## Surviving distinction

SDE-SQL answers:

> What probe does the LLM decide to run next?

ActiveProbeSQL must answer a stronger controlled question:

> Among explicitly enumerated candidate observations, which one has the largest expected value for resolving the current semantic decision per unit cost?

The scientific contribution cannot be the availability of probes. It must be the **selection objective and measurable information efficiency**.

---

# 3. Collision 2 — APEX-SQL already has hypothesis–verification exploration

APEX-SQL (2026) shifts Text2SQL from static schema reasoning to agentic exploration. It verbalizes hypotheses, profiles real data, validates column roles, and follows a hypothesis-verification loop during schema linking and SQL generation.

Reference: https://arxiv.org/abs/2602.16720

Thus “maintain hypotheses and verify them against the database” is also too broad.

## Surviving distinction

APEX-style exploration can be viewed as a policy encoded by directives and LLM reasoning. ActiveProbeSQL would make **experiment choice itself an optimization object**:

```text
current hypotheses + beliefs
            ↓
possible probes + costs + predicted outcomes
            ↓
expected posterior / expected decision utility
            ↓
choose probe
```

The benchmark must show that probe choice matters even when all methods receive the same probe interface and total budget.

---

# 4. Collision 3 — Bayesian experimental design with LLMs already exists

BED-LLM (2026) uses sequential Bayesian experimental design and expected information gain to select questions/queries to external sources. It constructs a probabilistic model from LLM predictive distributions and iteratively chooses observations that maximize EIG.

Reference: *BED-LLM: intelligent information gathering with LLMs and Bayesian experimental design*, 2026.

General-agent work on structured uncertainty also uses Expected Value of Perfect Information (EVPI) to select clarifying questions.

Reference: Suri et al., *Structured Uncertainty guided Clarification for LLM Agents*, ACL Findings 2026. https://aclanthology.org/2026.findings-acl.2028/

Therefore the paper cannot claim novelty for EIG/EVPI itself.

## Database-specific opportunity

A database gives unusually strong structure for experimental design:

- observations are executable queries;
- costs can be estimated from query plans/rows scanned;
- outcomes often have low-dimensional summaries;
- many hypotheses imply different distributions over probe outcomes;
- schema/constraints define which observations are legal;
- observations can be deterministically replayed;
- final success is executable and measurable.

This makes Text2SQL a particularly clean environment for studying **active information acquisition under realistic observation costs**.

---

# 5. Collision 4 — Agentic Data Elicitation already proposes controlled experimentation

The 2026 Agentic Data Environments agenda explicitly introduces **Agentic Data Elicitation (ADE)**: agents inspect systems, collect traces, form hypotheses, run controlled experiments, and turn latent signals into reusable artifacts.

Reference: Ang et al., *Agentic Data Environments*, 2026. https://arxiv.org/abs/2607.07397

The paper even names implicit table roles, undocumented relationships, and derived business logic as examples of latent data that experiments could surface.

This is conceptually very close.

## Surviving distinction

ADE is a systems/research agenda. ActiveProbeSQL can contribute a concrete formal task, benchmark, and algorithm:

```text
latent variable = correct SQL semantic plan
experiment = legal observation query
observation model = predicted probe-result distribution under each hypothesis
objective = expected final task utility - database/tool cost
```

A paper must make this formalization operational and compare it against strong Text2SQL exploration baselines.

---

# 6. Collision 5 — DPC/SpotIt/ParSEval distinguish programs using synthetic worlds

DPC, SpotIt, and ParSEval make “distinguish competing SQL semantics via carefully chosen database worlds” an occupied idea.

References:
- DPC: https://aclanthology.org/2026.acl-long.313/
- SpotIt: https://proceedings.iclr.cc/paper_files/paper/2026/hash/70e692da44c19710386648694e2b899b-Abstract-Conference.html
- ParSEval: DOI 10.14778/3749646.3749727

## Surviving distinction

These systems search/generate **alternative test databases** to expose program inequivalence, usually after candidate SQLs exist.

ActiveProbeSQL would operate on the **actual target database** before final semantic commitment:

```text
real D
 + uncertain intent/schema semantics
 + legal SELECT probes
 → choose observation
 → update belief
 → generate/finalize SQL
```

Synthetic distinguishing-world verification and real-world observation selection should be treated as complementary, not conflated.

---

# 7. Collision 6 — general active-reasoning RL is rapidly advancing

T3 (ICLR 2026 Oral) and AREW (ICML 2026) study agents that must actively seek information under partial observability and maintain belief states. They identify belief deviation and information self-locking as core learning failures.

Reference/code: https://github.com/unimpor/T3

This matters because a learned ActiveProbeSQL policy can fail for the same reason: an incorrect early belief may make later observations appear useless.

## Consequence for the proposed research

Do not begin with RL.

First establish the phenomenon with an **oracle observation model** or a controlled benchmark where information values are computable. Only then study learned belief/probe policies.

---

# 8. Exact problem formulation

Let:

```text
H = {h1, ..., hk}
```

be candidate semantic plans. A plan can encode:

- selected source tables;
- join interpretation;
- business measure definition;
- temporal semantics;
- entity/value grounding;
- result grain;
- aggregation semantics.

Maintain belief:

```text
b_t(h) = P(h | observations_1:t)
```

Candidate probe `p` has:

- legal executable SQL or metadata operation;
- estimated database cost `c_db(p)`;
- latency/tool cost `c_tool(p)`;
- outcome space `Y_p`;
- predicted likelihood `P(y | h, p)`.

Select:

```text
p* = argmax_p [ E_y V(b_{t+1}) - V(b_t) - λ c(p) ]
```

Possible value functions:

1. entropy reduction;
2. expected probability of selecting correct plan;
3. expected final task reward;
4. expected regret reduction;
5. EVPI-like decision value.

The strongest paper should compare these objectives rather than assume information gain is always task value.

---

# 9. What counts as a database probe?

A probe should be an **observation**, not a hidden attempt to solve the full task.

Candidate types:

## Schema probes

- inspect FK/key metadata;
- retrieve column statistics;
- fetch freshness/update timestamps;
- inspect table comments/ownership tags.

## Value probes

- `SELECT DISTINCT` samples;
- null prevalence;
- min/max/range;
- categorical support;
- regex/type consistency.

## Relational probes

- join overlap;
- fan-out/cardinality;
- orphan rate;
- duplicate rate at proposed grain;
- key uniqueness under filters.

## Temporal probes

- coverage by period;
- overlap of effective-date intervals;
- current-vs-history row counts;
- max event timestamps.

## Reconciliation probes

- compare invoice/payment/order-line totals;
- conservation/balance checks;
- cross-table population overlap.

## Execution probes

- run a restricted subquery;
- compare two local candidate fragments;
- inspect `EXPLAIN`/estimated cardinality.

---

# 10. Probe Compiler as a distinct subproblem

Free-form LLM probe generation creates confounds: a method may win because it writes better probe SQL, not because it selects better experiments.

Separate:

```text
semantic uncertainty
      ↓
probe type / experiment design
      ↓
Probe Compiler
      ↓
executable SQL
```

Example:

```text
uncertainty:
  join customer → order may fan out at customer grain

probe template:
  count multiplicity per customer after candidate join
```

This enables a clean study:

- oracle/hand-authored probe compiler + learned selector;
- learned compiler + oracle selector;
- fully learned system.

It also creates a reusable database diagnostic library.

---

# 11. Benchmark design: ambiguity with known discriminating experiments

A generic BIRD evaluation is insufficient because we do not know the ground-truth information value of probes.

Create controlled tasks where:

1. multiple semantic interpretations are deliberately plausible from schema/text alone;
2. the real database contains evidence that discriminates them;
3. many legal but low-value probes exist;
4. one or a small subset of probes is strongly discriminative;
5. observation costs vary.

Example family:

## Source-of-truth ambiguity

Tables:

```text
orders_legacy
orders_current
payments
invoices
```

Question:

> monthly booked revenue

Schema names/descriptions are intentionally ambiguous.

Useful evidence may be:

- data freshness;
- overlap with recent invoice IDs;
- reconciliation against payment population.

Irrelevant but tempting probes include random row samples from both tables.

This creates an observable probe-efficiency gap.

---

# 12. Oracle-gap-first pilot

Before developing a Bayesian controller, compute three ceilings:

## Oracle Probe Selector

Given all probe outcomes offline, choose the minimal subset needed to identify the correct plan.

## Random/Heuristic Selector

Choose probes uniformly or by standard heuristics.

## Free LLM Explorer

Give the same probe interface and budget to SDE/APEX-style reasoning.

If the oracle selector does not substantially beat free exploration in number/cost of observations, there is little room for a controller.

### Go/no-go criterion

A reasonable pilot threshold:

- at matched final accuracy, oracle/optimal selection should reduce observation cost by at least 20%; or
- at matched budget, oracle selection should improve semantic accuracy by at least 5 absolute points.

If neither occurs, stop.

---

# 13. Strong baselines

Required baselines include:

1. no exploration;
2. fixed metadata bundle;
3. SDE-SQL-style self-driven probes;
4. APEX-SQL-style hypothesis verification;
5. random legal probe;
6. heuristic probe priority;
7. entropy-reduction selector;
8. EVPI/expected-decision-value selector;
9. oracle selector.

If possible, separately evaluate:

- exact cost using actual warehouse bytes/latency;
- abstract unit cost for reproducibility.

---

# 14. Primary metrics

Do not report only execution accuracy.

Core metrics:

```text
semantic accuracy
execution accuracy
number of probes
database rows/bytes scanned
tool latency
total dollar-equivalent cost
entropy reduction per probe
accuracy per unit observation cost
```

Critical derived metric:

> **Probe Regret** = cost/utility gap to the oracle observation policy.

This is analogous to routing regret and gives a stable research target.

---

# 15. Hard failure modes worth studying

## F1. Belief collapse

The correct hypothesis gets low prior probability and is never recovered.

## F2. Correlated probes

The agent repeatedly buys nearly redundant observations.

## F3. Cheap-but-useless trap

Low-cost probes dominate the score without changing the final decision.

## F4. Expensive oracle trap

A highly discriminative query costs too much compared with asking the user or inspecting docs.

## F5. Observation model hallucination

The LLM incorrectly predicts what a probe would look like under a hypothesis.

## F6. Data coincidence

One database snapshot accidentally supports the wrong interpretation.

## F7. Query leakage

A “probe” becomes almost the full answer query, making the benchmark trivial.

## F8. Evidence-source conflict

Database values support one hypothesis while documentation supports another.

---

# 16. New derived directions

## P1. Ask-vs-Probe-vs-ReadDocs

Extend the action space from database probes to heterogeneous observations. Choose whether to ask the user, inspect metadata, search docs, or query data.

The novelty is not EVPI, but comparing human and machine evidence under a common decision-value/cost model.

## P2. Observation Caching

Cache probe results as reusable evidence artifacts with freshness/validity metadata. Research when a past observation remains valid after data drift.

## P3. Probe Causal Credit

After solving a task, counterfactually remove each observation and replay the decision to estimate whether that probe actually mattered.

This creates training labels for future probe policies.

## P4. Probe Distillation

Distill long expensive exploration trajectories into a short typed probe policy for recurring schema motifs.

## P5. Source-of-Truth Discovery Benchmark

Make authoritative-table selection the explicit hidden variable and evaluate optimal evidence acquisition.

## P6. Multi-Task Experimental Design

One probe may help many future queries on the same database. Optimize information gathering for a stream of tasks rather than one question.

## P7. Data-Quality-Aware Probing

Treat probe results themselves as noisy because the database can contain inconsistent or stale data. Choose redundant/orthogonal probes when evidence reliability is uncertain.

## P8. Safe Experimental Design

Add privacy/governance cost: some observations reveal sensitive values. Choose the most informative *policy-compliant* experiment.

## P9. Query-Plan-Aware Probe Cost

Use `EXPLAIN` to estimate scan/join cost before buying the observation. Optimize expected information per physical query cost.

## P10. Adaptive Probe Granularity

Choose between statistics, small samples, aggregates, and full scans based on how much precision the current decision needs.

---

# 17. Reviewer objections and required responses

## Objection: “SDE-SQL already uses probes.”

Response must be an equal-interface experiment showing that optimized probe *selection* reduces observation cost or improves accuracy under a fixed probe budget.

## Objection: “This is BED-LLM on databases.”

Response must demonstrate database-specific structure: executable relational observations, query-plan costs, relational hypotheses, deterministic replay, and Text2SQL outcomes.

## Objection: “APEX already does hypothesis verification.”

Response must isolate the value of formal experiment selection from generic hypothesis reasoning.

## Objection: “The benchmark is synthetic.”

Use a two-stage design: controlled benchmark for causal attribution, then BIRD/Spider2-derived tasks for external validity.

## Objection: “Your posterior is just another LLM confidence score.”

Include calibration analysis and oracle-likelihood ablations. The paper should remain informative even if learned beliefs are imperfect.

---

# 18. Revised judgment

The direction remains compelling, but only after substantial narrowing.

- “SQL probing”: occupied.
- “hypothesis verification”: occupied.
- “Bayesian experimental design with LLMs”: occupied.
- “agentic data elicitation”: proposed explicitly in 2026.

The remaining publishable object is:

> **cost-aware experiment selection over real relational observations for resolving Text2SQL semantic uncertainty.**

Revised score:

- Novelty: 4/5
- Feasibility: 4/5
- Scientific clarity: 5/5
- Collision risk: medium
- Best first artifact: controlled ambiguity/probe benchmark + oracle-gap analysis
- Implementation should begin only if the oracle probe gap is substantial.
