# Fresh Text2SQL + Data-Agent Frontier Survey and Batch Idea Factory (2026-08-24)

> Method: fresh literature-first spike. This memo intentionally does not inherit prior project idea rankings. It first audits the 2025–2026 frontier, marks occupied spaces, and then generates new research directions only where a distinct representation, objective, evaluation problem, systems primitive, or learning signal remains.

## 1. Research protocol

For each proposed direction:

1. Identify a concrete failure mode exposed by recent Text2SQL / data-agent work.
2. Identify the closest occupied mechanism.
3. State what **would not** be novel.
4. Define the surviving gap.
5. Give a falsifiable hypothesis and a minimal killer experiment.
6. Flag collision risk and engineering difficulty.

Ideas that are merely “add RAG”, “add more agents”, “add self-correction”, “add memory”, or “use more test-time compute” are rejected unless the new work changes the underlying scientific object.

---

# 2. Frontier audit: what is already occupied

## 2.1 Enterprise-scale Text2SQL is now an agentic workflow problem

Spider 2.0 contains 632 enterprise workflows, often with >1,000 columns, multiple SQL dialects, database documentation, and project-level code. Its original code-agent baseline solved only about 17% of tasks.

Reference: Lei et al., *Spider 2.0: Evaluating Language Models on Real-World Enterprise Text-to-SQL Workflows*, ICLR 2025 Oral. https://arxiv.org/abs/2411.07763

Implication: “large-schema Text2SQL” alone is not a novelty claim.

## 2.2 Dynamic interaction and CRUD are now explicit benchmark dimensions

BIRD-INTERACT evaluates conversational and open-ended agentic interaction with hierarchical knowledge, metadata, a user simulator, database exploration, error recovery, and full CRUD. GPT-5 reaches only 17% success in the open agentic setting reported in the paper.

Reference: Huo et al., *BIRD-INTERACT*, ICLR 2026 Oral. https://arxiv.org/abs/2510.05318

Implication: clarification, multi-turn execution, and CRUD need a sharper scientific question than “make Text2SQL interactive”.

## 2.3 Flexible exploration is heavily occupied

APEX-SQL uses hypothesis-verification loops, logical planning, data profiling, and empirical database exploration. FlexSQL allows schema/data inspection and verification at any reasoning point, diverse plans, SQL/Python execution, and plan-level backtracking.

References:
- Cao et al., *APEX-SQL: Talking to the data via Agentic Exploration for Text-to-SQL*, 2026. https://arxiv.org/abs/2602.16720
- Pham et al., *FlexSQL: Flexible Exploration and Execution Make Better Text-to-SQL Agents*, 2026. https://arxiv.org/abs/2605.02815

Implication: “let the agent explore the database” is a baseline, not a contribution.

## 2.4 Decomposition and software-engineering orchestration are occupied

AV-SQL introduces agent-generated CTE views as intermediate structures. DeepEye-SQL reframes Text2SQL as a software engineering workflow with N-version generation, deterministic tool-chain verification, revision, and confidence-aware candidate selection.

References:
- Pham et al., *AV-SQL*, 2026. https://arxiv.org/abs/2604.07041
- Li et al., *DeepEye-SQL: A Software-Engineering-Inspired Text-to-SQL Framework*, SIGMOD 2026. https://arxiv.org/abs/2510.17586

Implication: pipelines with planner/writer/critic/revisor roles need a fundamentally new control objective or evidence source.

## 2.5 Candidate selection without a gold execution oracle is advancing rapidly

DPC constructs a Minimal Distinguishing Database and cross-validates candidate SQL against an independent Python/Pandas solution, directly targeting the generation-selection gap.

Reference: Li et al., *DPC: Training-Free Text-to-SQL Candidate Selection via Dual-Paradigm Consistency*, ACL 2026. https://aclanthology.org/2026.acl-long.313/

Implication: “sample many SQLs and select with an LLM judge” is weak. New selection work should use new evidence or a new problem regime.

## 2.6 SQL refinement explicitly models errors

ErrorLLM trains dedicated error representations/tokens for implicit SQL error detection and guided refinement; MIRA decomposes confirmed corrections into reusable repair-memory items and verifies them against current database evidence before reuse.

References:
- Hong et al., *ErrorLLM*, 2026. https://arxiv.org/abs/2603.03742
- Liu et al., *MIRA: Evidence-Verified Repair Memory for Text-to-SQL Correction*, 2026. https://arxiv.org/abs/2608.06950

Implication: generic error taxonomies and error-fix memory are crowded.

## 2.7 Memory is moving from storage to measurable future value

AgentSM stores structured semantic programs from prior traces. Memo-SQL uses success and error-fix experience. August 2026 work on “memory crystallization” explicitly measures replay, cross-question retention, and held-out same-database transfer, finding that verified corrected queries improve future first-attempt accuracy and that database-specific content is the main operating ingredient.

References:
- Biswal et al., *AgentSM*, 2026. https://arxiv.org/abs/2601.15709
- Yang et al., *Memo-SQL*, 2026. https://arxiv.org/abs/2601.10011
- Wang et al., *From Test-Time Scaling to Reusable Memory: Measuring Crystallization in Text-to-SQL*, 2026. https://arxiv.org/abs/2608.07213

Implication: “add persistent memory” is no longer enough. The next questions are memory valuation, invalidation, causal credit, transfer, and governance.

## 2.8 Autonomous agent evolution is already demonstrated

RoboPhD autonomously evolves a Text2SQL agent implementation and generation instructions, using performance feedback and ELO-based selection. Agentar-Scale-SQL combines RL-enhanced reasoning, iterative refinement, and parallel candidate synthesis/selection.

References:
- Borthwick & Ash, *RoboPhD*, 2026. https://arxiv.org/abs/2601.01126
- Wang et al., *Agentar-Scale-SQL*, 2025. https://arxiv.org/abs/2509.24403

Implication: “evolve the SQL agent” is occupied unless the search space or scientific objective is new.

## 2.9 Debugging, optimization, and efficiency are becoming separate tasks

BIRD-CRITIC/SWE-SQL evaluates real SQL user issues across dialects and CRUD-like tasks. In June 2026 the BIRD team released Effi-SQL, which explicitly scores correctness together with execution speedup.

References:
- BIRD-CRITIC: https://bird-critic.github.io/
- SWE-SQL: https://arxiv.org/abs/2506.18951

Implication: SQL correctness and SQL operational efficiency should not be conflated.

## 2.10 Evaluation itself is in crisis

A VLDB 2026 study reports expert-audited annotation error rates of 52.8% on BIRD Mini-Dev and 62.8% on Spider 2.0-Snow and shows that corrections can substantially change scores and rankings.

Reference: Jin et al., *Pervasive Annotation Errors Break Text-to-SQL Benchmarks and Leaderboards*, PVLDB 2026. https://arxiv.org/abs/2601.08778

BADGER also argues that production-grade enterprise evaluation requires hybrid execution/structural evaluation plus agent-behavior metrics.

Reference: Serrao et al., *BADGER*, 2026. https://arxiv.org/abs/2606.02109

Implication: benchmark uncertainty is itself a research object.

## 2.11 AI-native SQL is a new and materially different regime

Spider 2.0-AIFunc contains 465 verified tasks using Snowflake AI functions. Strong proprietary models reach roughly 67–70% execution accuracy; importantly, traditional Text2SQL scaffolds such as elaborate schema retrieval do not transfer cleanly, and minimal agent setups can match or beat heavier frameworks.

Reference: Liu et al., *Spider 2.0-AIFunc*, 2026. https://arxiv.org/abs/2607.06229

Meanwhile, production platforms now expose AI predicates, semantic joins, ranking, classification, extraction, and generation directly inside SQL. AI calls can dominate query cost and latency by orders of magnitude.

References:
- Google BigQuery AI functions, 2025–2026.
- Snowflake Cortex AI Functions optimization, 2026.

Implication: Text2SQL is becoming Text2HybridSQL, where an “operator” may be stochastic, expensive, model-dependent, and semantically approximate.

## 2.12 Text2SQL is expanding into general data agents

The Data Agent Benchmark evaluates end-to-end questions across multiple heterogeneous DBMSs; the best reported frontier model reaches only 38% pass@1. DeepEye uses workflow-centric orchestration over databases, documents, files, and multimodal artifacts. Other enterprise work routes natural language through governed analytics APIs rather than raw SQL.

References:
- Ma et al., *Can AI Agents Answer Your Data Questions? A Benchmark for Data Agents*, 2026. https://arxiv.org/abs/2603.20576
- Li et al., *DeepEye: A Steerable Self-driving Data Agent System*, SIGMOD Demo 2026. https://arxiv.org/abs/2603.28889
- Singh et al., *Beyond Text-to-SQL: An Agentic LLM System for Governed Enterprise Analytics APIs*, 2026. https://arxiv.org/abs/2605.21027

Implication: a compelling future system may choose among SQL, Python, governed APIs, semantic layers, and AI-SQL operators rather than treating SQL as the only execution substrate.

## 2.13 Agent safety is moving into the data infrastructure

Data Flow Control (DFC) enforces policies over tuple-level data provenance inside the DBMS rather than trusting the LLM. Agentic Data Environments propose database/system branching and rollback as infrastructure for speculative agent actions. A 2026 security study finds substantial vulnerabilities across data agents, and real CVEs now exist for prompt-to-SQL execution without validation.

References:
- Summers & Wu, *Data Flow Control: Data Safety Policies for AI Agents*, 2026. https://arxiv.org/abs/2606.05679
- Ang et al., *Agentic Data Environments*, 2026. https://arxiv.org/abs/2607.07397
- Wang et al., *Data Agents Under Attack*, 2026. https://arxiv.org/abs/2606.08661

Implication: safe database agents should be designed around enforceable state/data-flow primitives, not prompt rules.

## 2.14 General agent research adds two important mechanisms

Structured-uncertainty clarification models uncertainty directly over tool parameters and uses Expected Value of Perfect Information (EVPI) to decide what to ask. Separately, STALE and Supersede show that long-term agents fail to invalidate outdated memory even when stronger models can understand the new evidence.

References:
- Suri et al., *Structured Uncertainty guided Clarification for LLM Agents*, ACL Findings 2026. https://aclanthology.org/2026.findings-acl.2028/
- Chao et al., *STALE*, 2026. https://arxiv.org/abs/2605.06527
- Patel, *Supersede*, 2026. https://arxiv.org/abs/2606.27472

Implication: value-of-information control and memory supersession are mature enough mechanisms to transfer carefully into data agents.

---

# 3. Batch idea factory

## Theme A — Adaptive agent architecture instead of one fixed scaffold

### A1. MetaSQL Router — choose the scaffold per task

**Gap.** Spider2-AIFunc reports that elaborate traditional Text2SQL scaffolds can lose to a minimal agent, while Spider 2.0 and BIRD-Interact reward more exploration. There is no reason one architecture should dominate all regimes.

**Mechanism.** A meta-controller chooses among a portfolio: direct generation, schema explorer, FlexSQL-like explorer, DPC-like verifier, memory-augmented corrector, interactive clarifier, or high-cost ensemble.

**Hypothesis.** Conditional scaffold selection matches or exceeds the best fixed scaffold while reducing median inference/tool cost.

**Killer experiment.** Train/tune the router on a mixture of BIRD, Spider2-Snow, Spider2-AIFunc, BIRD-Critic, and BIRD-Interact-Lite. Evaluate held-out task-regime transfer.

**Novelty risk:** low-medium. Generic model routing exists, but task-conditioned *agent architecture routing* across Text2SQL regimes appears open.

### A2. No-One-SQL-Agent Benchmark — scaffold generalization as the target

**Gap.** Papers optimize one benchmark and one scaffold. Cross-regime transfer of agent architecture is barely measured.

**Artifact.** Evaluate the same scaffold on classic SQL generation, enterprise large-schema SQL, interactive CRUD, SQL debugging, efficiency optimization, and AI-native SQL.

**Primary metric.** Regime-normalized regret relative to the best scaffold in each regime, plus cost.

**Hypothesis.** Many leaderboard-leading systems are brittle special-purpose policies rather than generally better SQL agents.

### A3. Budgeted Module Activation

Instead of always running every stage, each expensive module predicts its expected marginal value: more schema inspection, second plan, verifier, repair memory, synthetic distinguishing DB, stronger model, or user clarification.

**Scientific object:** marginal value of computation/tooling, not prompt design.

**Metric:** accuracy per dollar / latency / warehouse query.

### A4. Failure-Aware Escalation Ladder

Start with the simplest agent. Escalate only when calibrated signals indicate likely failure. Compare against always-heavy DeepEye-SQL/Agentar-style pipelines under equal total budget.

---

## Theme B — Active information acquisition over the real database

### B1. Discriminative ProbeSQL

**Closest occupied work.** APEX-SQL explores data through hypothesis-verification; FlexSQL can inspect data at any time; interactive Text2SQL asks users using information gain.

**Surviving gap.** Choose *database probes themselves* by expected discrimination between competing semantic plans.

Maintain candidate hypotheses such as:

```text
H1: revenue = invoice.total
H2: revenue = captured_payment - refund
H3: revenue = order_line.net_amount
```

Generate a cheap exploratory SQL whose possible outputs maximally change the posterior over H1/H2/H3.

**Killer experiment.** Equal probe budget against APEX heuristic exploration and FlexSQL free exploration. Primary endpoint: success per DB probe.

### B2. EVPI-Schema — value-of-information tool calling

Model uncertainty over schema/semantic slots: entity mapping, join edge, measure, time scope, result grain. Score `describe_table`, `sample_values`, `run_probe`, `search_docs`, and `ask_user` by expected task-value improvement minus cost.

This transfers the structured uncertainty/EVPI mechanism from general tool agents to Text2SQL, but includes *machine observations* as alternatives to human clarification.

### B3. Falsification-First SQL Exploration

Most agents seek supporting evidence. Reverse the objective: after proposing a plan, generate the cheapest test likely to falsify it. Accept only plans that survive a fixed falsification budget.

Possible probes: uniqueness, fan-out, temporal coverage, null prevalence, competing value distribution, alternative join support.

### B4. Probe Compiler

Compile one semantic uncertainty into a minimal executable diagnostic query. Example: “is `customer_id` one-to-one at report grain?” becomes a canonical duplicate/fan-out probe.

Research question: can a small library or learned compiler of semantic probes outperform free-form exploratory SQL?

### B5. Probe Portfolio Learning

Learn which diagnostic probes are most valuable for each failure signature from historical trajectories. Unlike repair memory, the memory object is *an experiment to run*, not an SQL fix.

---

## Theme C — Memory after AgentSM/MIRA/crystallization

### C1. Active Crystallization — learn what is worth storing

The August 2026 crystallization paper measures future value but does not make memory writing itself an explicit sequential decision problem.

**Idea.** After each expensive solved episode, predict future utility before storing it. Optimize under a fixed memory budget.

**Primary metric.** Future first-attempt improvement per stored byte/item.

### C2. Memory Causal Credit

For every retrieved memory item, estimate whether it actually improved or harmed the answer via controlled counterfactual replay with/without that item.

Use this to learn write/read policies and identify “popular but harmful” memories.

### C3. Anti-Memory for Exploration

Store structural dead ends and misleading evidence:

```text
schema region: billing_legacy
symptom: lexical match to revenue
outcome: deprecated source; never valid after 2025-01
```

Question: does remembering *where not to look* reduce enterprise exploration cost beyond positive memory?

### C4. DriftSQL Memory Benchmark

Evolve the same database over episodes: column rename, table split, new source-of-truth table, changed business metric, retained stale docs, altered dialect behavior.

Measure stale-memory harm, recovery speed, and unnecessary memory churn.

STALE/Supersede provide the adjacent-agent mechanism; Text2SQL contributes dependency-rich schema/business semantics and executable evaluation.

### C5. Motif Memory — cross-database abstraction

Convert database-specific memories into transferable motifs such as `slowly_changing_dimension_latest_row`, `bridge_table_dedup`, `event_status_history`, or `fact_dimension_star`.

Test whether motif memories transfer to entirely unseen schemas better than raw corrected SQL.

### C6. Memory Contamination Stress Test

Inject verified-but-now-stale, wrong-database, dialect-incompatible, or adversarial memories. Evaluate whether memory systems can detect and reject them before they corrupt correct SQL.

---

## Theme D — AI-native SQL as a new database language

### D1. StochasticSQL — semantics for nondeterministic AI operators

Traditional SQL expects deterministic relational operators. AI functions can vary across repeated executions, model versions, temperature/configuration, and prompt rewrites.

**Idea.** Extend Text2SQL evaluation from one result set to a *distributional contract*: stability, agreement rate, confidence interval, and semantic tolerance.

**Killer experiment.** Repeat Spider2-AIFunc tasks across time/model versions and quantify how often “correct SQL” does not imply stable answers.

### D2. Model-as-Operator Query Optimizer

Treat each AI model endpoint as a physical operator with a cost/latency/quality/selectivity profile. The planner chooses model + operator order under an SLA.

This turns AI-SQL generation into classical physical query optimization with stochastic operator costs.

### D3. AI-Predicate Reordering with Semantic Selectivity

Production platforms already optimize AI filters because LLM calls dominate cost. Research a Text2SQL agent that jointly generates predicates and predicts semantic selectivity to minimize expected AI calls without changing the requested semantics.

### D4. Proxy-First AI-SQL

Use cheap proxy models or embeddings to prefilter rows, call expensive AI operators only near the decision boundary, and preserve a target recall/precision guarantee.

Research target: cost reduction at fixed end-to-end semantic quality.

### D5. AI-SQL Reproducibility Contracts

A generated AI-SQL query should carry its model endpoint, prompt/rubric version, evaluation tolerance, retry policy, and stability test. Evaluate reproducibility under endpoint upgrades.

### D6. Approximate AI-SQL

Adaptively sample rows or AI calls until an answer-level confidence target is reached, analogous to approximate query processing but with semantic model uncertainty.

### D7. Agent Complexity Inversion

Spider2-AIFunc reports that a minimal agent can outperform elaborate traditional Text2SQL agents. Build a systematic study of *when agentic complexity hurts* due to extra opportunities for function/parameter corruption.

Possible predictor: ratio of relational complexity to AI-operator complexity.

---

## Theme E — Polyglot data agents beyond SQL

### E1. Execution-Substrate Router

Given an analytical intent, choose among:

- raw SQL,
- Python/Pandas,
- governed enterprise API,
- semantic/metrics layer,
- AI-native SQL,
- document retrieval + structured computation.

FlexSQL covers SQL/Python; governed API work and AI-SQL work cover separate substrates. The new question is unified substrate selection under correctness, governance, and cost.

### E2. Data-Source Locality Planner

For multi-source tasks, decompose computation so sensitive data stays local and only safe aggregates cross boundaries.

Evaluate on DAB-style heterogeneous workloads with permissions and transfer costs.

### E3. Cross-Source Evidence Graph

Every final analytical claim carries edges to the SQL query, source database rows/aggregates, documents, API calls, and transformations that support it.

Unlike a chain-of-thought trace, this is an executable provenance graph usable for audit and partial recomputation.

### E4. Cross-System Entity Resolution Agent

A major hidden difficulty in DAB-style workloads is that the same real-world entity has inconsistent identifiers across systems. Make entity alignment a first-class latent variable with explicit uncertainty and validation probes.

### E5. Workflow Re-Optimizer for Data Agents

DeepEye has a workflow engine and topological optimization. Push further: learn runtime rewrites of the agent DAG from intermediate cardinalities, tool latency, cache hits, and discovered independence.

The evaluation target is workflow makespan/cost at fixed answer correctness.

---

## Theme F — Safe read/write database agents

### F1. Branch-and-Verify CRUD

BIRD-Interact now makes CRUD realistic, while Agentic Data Environments argue for branchable state as agent infrastructure.

**Mechanism.** Execute candidate writes on an isolated database branch, run postconditions/invariants, compare relational delta, then either merge or discard.

**Killer experiment.** BIRD-Interact CRUD with destructive/error-prone variants. Measure task success and prevented bad mutations.

### F2. Semantic Delta Contracts

Compile a natural-language write request into an expected relational delta before generating SQL:

```text
rows_inserted: exactly 1 into subscriptions
rows_updated: <= 1 in customer_status
must_preserve: total_paid_amount
must_not_touch: audit_log
```

After speculative execution, compare actual delta with the contract.

This is stronger than AST allowlists because it verifies *state effect*.

### F3. DFC-Aware Text2SQL

Data Flow Control can enforce derivation policies deterministically. Study generation/planning when the agent knows its query must satisfy DFC constraints.

Question: can the agent reformulate an analytical task into a policy-compliant equivalent rather than simply being blocked?

### F4. Least-Privilege Exploration

Current evaluation focuses on final answer or SQL. Score the entire trajectory by how many sensitive tables, columns, or rows it touches unnecessarily.

Objective: task success with minimum data exposure.

### F5. Agent Data-Surface Privacy Benchmark

The 2026 privacy survey notes that leakage can occur through queries, intermediate results, memory, and inter-agent messages. Build a Text2SQL/data-agent benchmark where one policy governs *all* surfaces simultaneously.

### F6. Policy-Preserving Decomposition

Before any LLM sees sensitive cells, push filters, aggregation, redaction, and k-anonymity-like constraints into deterministic subqueries. Let the model reason only over policy-compliant intermediate views.

### F7. Capability/Effect Types for SQL Tools

Annotate tools and plan nodes with effects such as `READ[table]`, `WRITE[table]`, `EXPOSE[PII]`, `EXTERNAL_CALL`, `IRREVERSIBLE`. Reject or branch workflows whose effect composition violates policy.

---

## Theme G — Evaluation and benchmark science

### G1. Label-Uncertainty-Aware Leaderboards

Given the 2026 annotation-error findings, report agent scores as intervals/posteriors over uncertain labels rather than a single exact percentage.

Use expert-audited subsets to estimate task-level annotation reliability and propagate it into ranking uncertainty.

### G2. Ranking Stability as a Metric

Evaluate whether a method’s claimed advantage survives:

- corrected annotations,
- database snapshot changes,
- model backend changes,
- equalized test-time compute,
- multiple SQL dialects.

A 1-point gain that reverses under small benchmark corrections should be treated differently from a robust 1-point gain.

### G3. Adaptive Evaluator via Distinguishing Worlds

When candidate and reference SQL disagree, synthesize only enough database instances to determine whether the disagreement is semantically meaningful. This turns evaluation into active equivalence testing rather than blind exact-match or one-instance execution.

Closest collision: DPC uses a Minimal Distinguishing Database for candidate selection; SynSQL generates synthetic DBs for robust evaluation. Novelty would require an evaluator optimized for *annotation adjudication cost and uncertainty*.

### G4. Database-Snapshot Robustness Benchmark

Keep schema/question fixed while changing the legal data instance over time. Correct semantics should survive ordinary data drift. Score how often agent decisions were accidentally tailored to one snapshot.

### G5. Benchmark Shortcut Audit

Train probes to predict benchmark labels using non-semantic shortcuts: table names, evidence phrasing, schema size, task source, annotation artifacts. Generate adversarial splits that break these shortcuts.

### G6. Cost-Normalized Text2SQL Evaluation

Report a Pareto frontier over model tokens, tool calls, warehouse scan cost, AI-function calls, latency, and human turns. Effi-SQL measures final-query speed; this measures *total cost of obtaining the answer*.

---

## Theme H — User interaction beyond oracle clarification

### H1. Non-Oracle User Simulator

BIRD-Interact’s simulator is useful but real users can be incomplete, uncertain, contradictory, or unaware of schema/business definitions.

Build user personas with bounded knowledge and confidence. The agent must decide whether to trust, verify, or challenge an answer.

### H2. Ask-vs-Inspect-vs-Execute EVPI

At each uncertainty, compare the value of:

- asking the user,
- inspecting schema,
- sampling data,
- running a verification query,
- reading docs,
- proceeding directly.

Interactive Text2SQL EIG optimizes clarification; structured uncertainty optimizes tool-parameter questions. The new contribution is a unified observation-selection policy across human and machine evidence.

### H3. Human-AI SQL Debugging Policy

BIRD-CRITIC reports a large gap between humans without and with GenAI tools. Instead of maximizing autonomous success, optimize *team success per human minute*.

Agent learns when to autonomously fix, when to present two hypotheses, and when to ask a database expert for one specific decision.

### H4. Contestable SQL

For high-impact queries, the system surfaces a compact set of semantic commitments before execution: metric definition, grain, join path, time policy, and source of truth. Users can contest one commitment without rewriting the whole request.

Research question: does structured contestability reduce silent semantic failures with less interaction than free-form clarification?

---

## Theme I — Enterprise metadata, evidence, and trust

### I1. Conflicting-Metadata Text2SQL

Construct tasks where documentation, FK metadata, sample values, historical queries, and catalog descriptions disagree. Each evidence source has an unknown reliability profile.

The agent must learn source trust rather than assuming retrieved context is correct.

### I2. Evidence Provenance Ledger

Every semantic decision records supporting observations and their source/version. If a source changes, only dependent decisions are invalidated.

This differs from ordinary memory because the object is a dependency graph of *why the current SQL is believed*, not a retrieval store.

### I3. Source-of-Truth Discovery

Enterprise schemas often contain legacy and replacement tables. Build a task where the correct answer depends on identifying the current authoritative data product from usage, freshness, governance tags, and data consistency—not lexical similarity.

### I4. Historical-Query Semantics Induction

Instead of retrieving nearest past SQL, mine many historical queries to infer stable organization-level rules and exceptions, then verify those rules on held-out workload behavior.

Collision risk: memory work is crowded; novelty requires cross-trajectory abstraction and explicit rule induction rather than example retrieval.

---

## Theme J — Learning and self-evolution after RoboPhD

### J1. Evolve Conditional Scaffolds, Not Monolithic Agents

RoboPhD evolves an agent implementation. New search space: a conditional computation graph where modules activate based on task state. Evolution optimizes both accuracy and expected cost.

### J2. Verifier Discovery

Given failure clusters, autonomously synthesize *new executable probes/invariants* that predict failure, rather than new prompts. Retain a verifier only if it improves held-out detection under controlled false-positive rate.

### J3. Adversarial Database Curriculum

Generate database instances specifically targeted at the current agent’s semantic weaknesses, analogous to adversarial training. Unlike generic SQLForge/SynSQL synthesis, the generator is closed-loop and failure-conditioned.

### J4. Distill Test-Time Compute into a Tool Policy

Expensive agent trajectories reveal which observations were actually necessary. Train a cheaper policy to predict the minimal useful sequence of tool calls on future tasks.

This goes beyond crystallizing corrected SQL into memory; it distills the *information-acquisition policy*.

### J5. Research-Claim Evolution

Let autonomous research search over hypotheses such as “candidate diversity matters only when schema ambiguity exceeds X” rather than searching prompt strings. Each mutation changes one causal claim and generates the minimal experiment needed to falsify it.

---

# 4. High-priority shortlist after collision screening

The following directions look strongest after accounting for the 2026 occupied space.

| Rank | Direction | Why it survives | Novelty | Feasibility | Best venue flavor |
|---|---|---|---:|---:|---|
| 1 | MetaSQL Router / No-One-SQL-Agent | AIFunc shows complex scaffolds can hurt; field lacks cross-regime adaptive architecture | 5 | 4 | ACL/EMNLP/SIGMOD |
| 2 | Discriminative ProbeSQL | Exploration exists, but probe selection by hypothesis discrimination is sharper | 5 | 4 | ACL/ICLR/SIGMOD |
| 3 | StochasticSQL / AI-SQL contracts | AI-native SQL creates genuinely new nondeterministic operator semantics | 5 | 4 | SIGMOD/VLDB |
| 4 | Branch-and-Verify CRUD + Semantic Delta Contracts | BIRD-Interact + Agentic Data Environments make read/write safety experimentally tractable | 5 | 3 | SIGMOD/VLDB |
| 5 | Total-Cost-of-Answer benchmark | Current work separately measures final SQL efficiency, token cost, or tool usage | 4 | 5 | VLDB/benchmark |
| 6 | DriftSQL Memory Benchmark | Memory is hot, but invalidation under executable schema/business drift remains under-tested | 4 | 5 | ACL/ICLR |
| 7 | Active Crystallization | Latest memory paper measures future value; next natural step is optimizing write decisions | 4 | 4 | ACL/ICLR |
| 8 | Data-Surface Privacy Benchmark | Survey explicitly identifies missing cross-surface privacy evaluation | 5 | 3 | CCS/USENIX/SIGMOD |
| 9 | Execution-Substrate Router | SQL/Python/API/AI-SQL are currently studied separately | 5 | 3 | SIGMOD/VLDB |
| 10 | Non-Oracle Interactive SQL | Existing user simulators usually act as reliable information sources | 5 | 4 | ACL/CHI/ICLR |
| 11 | Conflicting-Metadata Text2SQL | Modern agents consume many evidence sources but usually assume retrieval truthfulness | 4 | 5 | ACL/EMNLP |
| 12 | Verifier Discovery | Moves autonomous evolution from prompts/scaffolds to executable scientific checks | 5 | 3 | ICLR/NeurIPS |

---

# 5. Four paper programs that can each generate multiple papers

## Program P1 — Adaptive SQL Agents

Core thesis:

> There is no universally best Text2SQL scaffold; reliable systems should conditionally allocate architecture and computation to the task regime and uncertainty state.

Possible sequence:

1. No-One-SQL-Agent benchmark.
2. MetaSQL scaffold router.
3. Budgeted module activation.
4. Test-time-compute-to-policy distillation.

## Program P2 — Experimental Text2SQL

Core thesis:

> Database interaction should be treated as experimental design: choose the cheapest observation that most discriminates competing semantic hypotheses.

Possible sequence:

1. Discriminative ProbeSQL.
2. Probe compiler.
3. Falsification-first reasoning.
4. Learned probe portfolios.

## Program P3 — AI-Native Query Agents

Core thesis:

> Once LLM calls become SQL operators, Text2SQL becomes query planning under stochastic semantics, cost, latency, and model drift.

Possible sequence:

1. StochasticSQL benchmark.
2. Model-as-operator optimizer.
3. Proxy-first AI-SQL.
4. Approximate AI-SQL with confidence targets.
5. Reproducibility contracts.

## Program P4 — Safe Stateful Data Agents

Core thesis:

> Reliable autonomous data agents need database/system primitives that make speculative execution, provenance constraints, and state effects verifiable before committing.

Possible sequence:

1. Branch-and-Verify CRUD benchmark.
2. Semantic Delta Contracts.
3. DFC-aware planning.
4. Least-privilege exploration and cross-surface privacy.

---

# 6. Ideas explicitly rejected as too occupied in their naive form

Do **not** treat the following as standalone contributions without a much sharper mechanism:

- “multi-agent Text2SQL”;
- “schema RAG”;
- “execution feedback + repair”;
- “hypothesis-verification exploration”;
- “agent-generated CTE decomposition”;
- “software-engineering pipeline for SQL generation”;
- “store successful SQL examples in memory”;
- “store error-fix pairs in memory”;
- “sample many SQLs and majority vote”;
- “LLM judge chooses the best SQL”;
- “test-time scaling improves Text2SQL”;
- “autonomous evolution improves an SQL agent”;
- “ask clarification when uncertain”;
- “optimize final SQL runtime”;
- “use governed APIs instead of raw SQL”;
- “generate synthetic Text2SQL training data”.

Every item above has strong 2025–2026 prior art. A new paper needs a new scientific axis.

---

# 7. Suggested next spike sequence

Without implementing a large system yet, the strongest next literature/novelty passes are:

1. **MetaSQL / scaffold routing:** audit adaptive-computation and model-routing literature, then search for any Text2SQL-specific architecture router collision.
2. **Discriminative ProbeSQL:** audit active experiment design, query-by-committee, Bayesian experimental design, and database diagnosis literature.
3. **StochasticSQL:** audit probabilistic databases, approximate query processing, AI/UDF query optimization, semantic caching, and current AI-function platform semantics.
4. **Branch-and-Verify CRUD:** audit database branching, transaction/saga semantics, state-delta verification, and agentic data environments.
5. **Privacy:** audit information-flow control and data-flow-control benchmarks for agent trajectories.

Only after these collision checks should engineering begin.

---

# 8. Key references used in this fresh pass

- Spider 2.0: https://arxiv.org/abs/2411.07763
- BIRD-INTERACT: https://arxiv.org/abs/2510.05318
- APEX-SQL: https://arxiv.org/abs/2602.16720
- FlexSQL: https://arxiv.org/abs/2605.02815
- AV-SQL: https://arxiv.org/abs/2604.07041
- DeepEye-SQL: https://arxiv.org/abs/2510.17586
- DPC: https://aclanthology.org/2026.acl-long.313/
- ErrorLLM: https://arxiv.org/abs/2603.03742
- AgentSM: https://arxiv.org/abs/2601.15709
- Memo-SQL: https://arxiv.org/abs/2601.10011
- MIRA: https://arxiv.org/abs/2608.06950
- Memory crystallization: https://arxiv.org/abs/2608.07213
- RoboPhD: https://arxiv.org/abs/2601.01126
- Agentar-Scale-SQL: https://arxiv.org/abs/2509.24403
- BIRD-CRITIC / SWE-SQL: https://arxiv.org/abs/2506.18951
- Annotation-errors study: https://arxiv.org/abs/2601.08778
- BADGER: https://arxiv.org/abs/2606.02109
- Spider 2.0-AIFunc: https://arxiv.org/abs/2607.06229
- Data Agent Benchmark: https://arxiv.org/abs/2603.20576
- DeepEye: https://arxiv.org/abs/2603.28889
- Governed enterprise analytics APIs: https://arxiv.org/abs/2605.21027
- Data Flow Control: https://arxiv.org/abs/2606.05679
- Agentic Data Environments: https://arxiv.org/abs/2607.07397
- Data Agents Under Attack: https://arxiv.org/abs/2606.08661
- Structured Uncertainty guided Clarification: https://aclanthology.org/2026.findings-acl.2028/
- STALE: https://arxiv.org/abs/2605.06527
- Supersede: https://arxiv.org/abs/2606.27472
- Business Logic-Driven Text-to-SQL Data Synthesis: https://arxiv.org/abs/2601.14518
- SQLForge: https://arxiv.org/abs/2505.13725
- SQL-Factory: https://arxiv.org/abs/2504.14837
- SQLord: https://arxiv.org/abs/2507.10629
- Semantic Caching for OLAP: https://arxiv.org/abs/2602.19811
- BranchBench: https://arxiv.org/abs/2604.17180
