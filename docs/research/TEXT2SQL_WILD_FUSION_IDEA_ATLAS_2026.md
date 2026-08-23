# Text2SQL Wild Fusion Idea Atlas (2026)

> Date: 2026-08-23  
> Goal: deliberately move beyond the previous Text2SQL idea map. This volume avoids re-centering the already-covered CEGIS, MCTS, generic multi-agent debate, metamorphic verification, typed IR, standard RAG, causal perturbation, abstract interpretation, and conventional evolutionary search.  
> Rule: borrow mechanisms, not buzzwords. Every imported concept should change representation, search, verification, learning, resource allocation, safety, or evaluation in a falsifiable way.

---

## 0. A machine for generating more ideas

A useful way to keep the research program from converging too early is to generate ideas compositionally.

Pick one item from each axis:

### Source mechanism

- thermodynamics / statistical mechanics
- quantum mechanics
- neuroscience / cognitive science
- immunology
- ecology / evolution
- developmental biology
- chemistry / retrosynthesis
- robotics / navigation
- operating systems
- computer networks
- SRE / fault engineering
- economics / mechanism design
- law / argumentation
- accounting / audit
- cryptography / privacy
- category theory / topology
- probabilistic programming
- linguistics / pragmatics
- education / tutoring
- architecture / urban planning

### Text2SQL bottleneck

- schema discovery
- semantic ambiguity
- join-path selection
- result grain
- aggregation
- temporal semantics
- value grounding
- long-horizon tool use
- repair
- confidence / abstention
- safety
- cross-dialect transfer
- recurring enterprise schemas
- cost / latency
- robustness to stale or malicious metadata

### Intervention point

- representation
- search
- memory
- verifier
- tool policy
- controller
- training data
- self-improvement
- benchmark generation
- confidence estimation
- resource scheduling
- final-answer certificate

### Scientific objective

- task success
- silent semantic error
- calibration
- cost
- latency
- robustness
- transfer
- interpretability
- safety
- sample efficiency

This gives thousands of combinations. The rest of this document instantiates the most interesting ones.

---

# 1. Physics, thermodynamics, and quantum-inspired ideas

## 1.1 NoetherSQL — correctness from invariances

Noether's theorem connects symmetries to conserved quantities. For SQL, many task semantics should be invariant under transformations that do not change meaning.

Candidate symmetries:

- table / column presentation order
- harmless alias renaming
- equivalent units after explicit conversion
- equivalent date encodings
- equivalent schema views
- permutation of irrelevant tables

Instead of treating robustness tests as an afterthought, define a **symmetry group of the task** and require the agent's relational semantics to transform equivariantly.

### Hypothesis

Agents trained or selected by symmetry consistency have lower silent semantic error than agents selected by execution success alone.

### Strong artifact

A benchmark that ships each task with a set of legal semantic symmetries and automatically scores equivariance.

---

## 1.2 Gauge-fixed SQL

Physics often admits multiple mathematically equivalent descriptions related by gauge transformations. SQL has an analogous nuisance: many schemas and SQL programs represent the same underlying business relation.

Define a canonical `gauge-fixed` semantic representation:

```text
entity roles
measure roles
join topology
result grain
filter semantics
```

All equivalent aliases, view names, and SQL surface forms map to the same gauge-fixed state.

### Research question

Does forcing reasoning in a canonical semantic gauge improve cross-schema and cross-dialect transfer?

This is more ambitious than simple AST normalization because the canonicalization target is business semantics, not syntax.

---

## 1.3 Renormalization-Group Schema Reasoning

Renormalization works by changing resolution while preserving the important macroscopic structure.

Treat a huge enterprise schema at several scales:

```text
organization
  -> business domain
    -> subject area
      -> table cluster
        -> table
          -> column
```

The agent first reasons over coarse variables, then progressively `flows` toward fine-grained schema details only where needed.

### Novel angle

Learn which semantic properties are stable across scale and which only emerge at fine resolution.

### Metric

Accuracy versus inspected schema entropy at each resolution level.

---

## 1.4 Free-Energy SQL Agent

Borrow the active-inference idea that an agent balances model fit with complexity / action cost.

Define a free-energy-like objective:

```text
F(plan) = semantic_surprise
        + beta * reasoning_cost
        + gamma * DB_cost
        + delta * interaction_cost
        + rho * safety_risk
```

The agent does not merely maximize confidence. It minimizes a structured objective over uncertainty and action cost.

### Experiment

Compare fixed tool policies against a controller that dynamically trades schema inspection, user questions, execution probes, and verification.

---

## 1.5 Semantic Annealing

Simulated annealing escapes local minima by accepting worse moves early and becoming conservative later.

A SQL agent could deliberately maintain high semantic flexibility early:

- swap candidate entity mappings
- alter join topology
- revise grain
- reinterpret metric definitions

Then gradually freeze decisions as evidence accumulates.

### Key difference from iterative repair

Repair usually edits after failure. Semantic annealing explicitly **prevents premature commitment** before failure.

### Research question

Do hard Text2SQL errors arise from early semantic freezing more than from poor final decoding?

---

## 1.6 Hamiltonian Meet-in-the-Middle Planning

Build two reasoning trajectories:

**Forward:** user intent -> candidate operations -> candidate relations.  
**Backward:** desired output schema / grain -> required measures -> required precursor relations.

The system searches for a low-energy meeting point between the two trajectories.

### Example

Forward path says `customer -> orders -> refunds`. Backward path says the output requires `one row per customer` and `net revenue`, implying refunds must be pre-aggregated. Their disagreement identifies the semantic bottleneck.

This resembles bidirectional search, but the state is a typed semantic contract rather than a graph node.

---

## 1.7 Phase-Transition Benchmarking

Instead of reporting only average accuracy, search for **complexity phase transitions**.

Increase one dimension at a time:

- schema size
- decoy-table density
- join-graph cycle count
- ambiguity count
- conversation length
- number of temporal clauses

Measure the point at which a method collapses.

### Paper framing

> When Does Agentic Text2SQL Break? Empirical Phase Transitions in Schema and Reasoning Complexity.

This could reveal qualitatively different regimes where entirely different architectures are required.

---

## 1.8 Quantum-Inspired Hypothesis Superposition

Do not collapse to one schema interpretation too early. Maintain a weighted set of mutually incompatible hypotheses:

```text
H1: revenue = invoices.net_amount      p=.46
H2: revenue = orders.total_amount      p=.38
H3: revenue = ledger.recognized_rev    p=.16
```

A tool call is chosen to maximally separate the hypotheses; evidence then reweights them.

The quantum analogy is only useful if it changes the algorithm: maintain incompatible interpretations explicitly and delay collapse until observation.

---

# 2. Neuroscience and cognitive-science ideas

## 2.1 PredictiveCodingSQL

Predictive coding alternates top-down predictions and bottom-up prediction errors.

For Text2SQL:

1. intent model predicts which entities, measures, and relations should exist;
2. schema evidence reports mismatch;
3. only the mismatch is propagated upward;
4. the semantic hypothesis is revised.

Example:

```text
Prediction: customers has a region attribute.
Observation: it does not.
Prediction error: region exists only on customer_addresses.
Revision: geography requires address-history semantics.
```

### Hypothesis

Prediction-error-driven exploration uses fewer metadata calls than flat retrieval while improving rare-schema adaptation.

---

## 2.2 Hippocampal SQL Replay

Human memory consolidates experience through replay. Instead of storing a static failure database, replay old trajectories offline and **re-solve them under changed conditions**:

- renamed schema
- new dialect
- missing documentation
- additional decoy tables
- tighter token budget

The replay system extracts semantic lessons that survive transformations.

### Distinction

Memory becomes an active consolidation process, not retrieval of frozen examples.

---

## 2.3 Dreaming SQL Agents

During idle time, generate synthetic `dreams` by recombining real failure fragments:

- one task's temporal ambiguity
- another task's many-to-many schema
- another task's misleading documentation

Then solve the synthetic tasks and keep only those whose verifier can establish a meaningful semantic distinction.

### Research question

Can unsupervised offline dream generation improve robustness to unseen combinations of failure modes?

---

## 2.4 Basal-Ganglia Gating for Tools and Memory

The basal ganglia is often modeled as a gating system that selects which actions or representations enter working memory.

Create a small controller that gates:

- schema retrieval
- sample-row inspection
- business glossary memory
- expensive verifier
- stronger LLM
- user clarification

Each gate has explicit cost and expected value.

### Experiment

Compare a learned gating controller with a monolithic LLM allowed to call everything.

---

## 2.5 Global-Workspace SQL

Instead of every specialist sharing full context, maintain a small global workspace. Specialists privately process different evidence sources and only broadcast **high-surprise, high-relevance findings**.

Possible specialists:

- DDL specialist
- documentation specialist
- value-distribution specialist
- temporal specialist
- governance specialist

The workspace has a strict token capacity.

### Hypothesis

Sparse broadcasting reduces correlated hallucination and context bloat compared with full-context multi-agent architectures.

---

## 2.6 Cerebellar Residual Repair

The cerebellum is often associated with fast predictive error correction.

Train a lightweight repair module on the **residual** between a candidate semantic plan and verifier feedback:

```text
candidate plan + structured error -> minimal semantic patch
```

Examples:

- change `COUNT` -> `COUNT DISTINCT`
- pre-aggregate bridge table
- switch inner -> left join
- move filter pre/post aggregation

The large model handles planning; the small repair model handles recurring micro-errors cheaply.

---

## 2.7 Schema Place Cells and Grid Cells

Robotic and animal navigation use compact spatial representations. Build embeddings that encode a schema as a navigable semantic space:

- landmarks = canonical business entities
- distance = relational / semantic traversal cost
- bridge tables = corridors
- ambiguous columns = intersections

The agent learns `where it is` in schema space while interpreting the question.

### Evaluation

Cross-database navigation to semantically analogous structures, even when names differ completely.

---

## 2.8 Working-Memory Chunking of Database Motifs

Experts do not see 40 independent tables; they recognize motifs:

- star schema
- slowly changing dimension
- event log
- bridge table
- ledger
- funnel
- cohort
- status history

Compress schema neighborhoods into named chunks, then reason over chunks before expanding them.

### Research question

Does motif chunking increase effective context length and improve transfer to very large schemas?

---

## 2.9 Metacognitive Failure Forecasting

Before generating SQL, predict a distribution over likely failure modes:

```text
wrong_join_cardinality: .31
metric_ambiguity: .27
temporal_boundary: .19
value_grounding: .08
```

Use that distribution to choose specialized checks.

### Important evaluation

Score the forecast itself. A useful metacognitive system should be calibrated about **how it is likely to fail**, not merely confident about success.

---

# 3. Immunology, ecology, evolution, and chemistry

## 3.1 ImmuneSQL — negative selection

The immune system learns to reject dangerous patterns without enumerating every safe state.

Build a library of semantic `detectors` for dangerous SQL patterns:

- unguarded fan-out
- ambiguous latest-record logic
- null-sensitive `NOT IN`
- non-additive measures being summed
- snapshot metrics joined to events
- destructive writes outside transaction boundaries

Train / evolve detectors primarily from failures, not successful examples.

### Hypothesis

Negative-selection detectors generalize to unseen tasks better than retrieval of positive exemplars.

---

## 3.2 Clonal Plan Expansion

When a candidate plan appears promising, clone it and mutate only uncertain semantic loci.

Example:

```text
stable: tables, time window, result grain
uncertain: join edge between accounts and customers
```

Generate a family of local variants around that edge, not whole independent plans.

This yields a structured population search with much higher sample efficiency than naive self-consistency.

---

## 3.3 Danger-Theory Verification

Do not verify every query equally. Escalate only when danger signals appear:

- high fan-out estimate
- ambiguous metric name
- cross-domain join
- PII access
- mutation query
- contradictory docs
- repeated repair failure

### Research question

Can danger-triggered verification match exhaustive verifier accuracy at a fraction of cost?

---

## 3.4 Immune Memory for Rare Traps

Store memory cells for low-frequency, high-impact failures such as:

- slowly changing dimension leakage
- fiscal-calendar mismatch
- late-arriving fact tables
- currency conversion at wrong date
- multiple `current` records

Unlike nearest-neighbor retrieval, memory cells are compact **recognizers** that activate when a new task exhibits the same structural pathology.

---

## 3.5 Ecological Niches for Agent Populations

A single best SQL agent may not exist. Maintain a population with explicit niches:

- star-schema specialist
- event-log specialist
- finance-ledger specialist
- healthcare-temporal specialist
- warehouse-cost specialist

A router assigns tasks, but agents also compete / specialize over time.

### Research question

Does ecological specialization outperform one globally optimized agent at equal aggregate parameter / token budget?

---

## 3.6 Biodiversity as an Ensemble Objective

Standard ensembles optimize individual accuracy. Add an explicit **semantic diversity** term so members fail differently.

Reward diversity over:

- schema paths
- decomposition style
- evidence source
- semantic assumptions

Then select a minimal population that maximizes coverage of distinct failure modes.

This treats ensemble design as ecosystem design rather than repeated sampling.

---

## 3.7 SQL Retrosynthesis

Chemists solve synthesis problems backward from a target molecule to precursor molecules.

Do the same for relational answers.

Start from the desired output contract:

```text
columns: customer_id, net_revenue
result grain: customer
ordering: net_revenue desc
```

Ask recursively:

- what precursor relation can produce `net_revenue`?
- what precursor produces refunds per order?
- what precursor produces valid orders in the time window?

Only after the backward synthesis tree is complete is SQL emitted.

### Why this is different

Most planners go forward from tables. Retrosynthesis starts from the semantic product and derives necessary precursors.

---

## 3.8 Catalytic SQL Agents

A catalyst changes reaction pathways without being consumed. Some tools may play the same role: a small amount of external structure can drastically simplify planning.

Candidate catalysts:

- semantic-layer metadata
- a single verified join path
- one exemplar query from an analyst
- a glossary definition
- one sample output row

### Research question

Which minimal `catalyst` gives the largest improvement per token / human effort?

This frames human-in-the-loop design as catalyst discovery, not generic supervision.

---

## 3.9 Stoichiometric Query Checks

In chemistry, reactions obey conservation laws. Many business queries also obey balance-like constraints.

Examples:

```text
opening_balance + inflow - outflow = closing_balance
orders = fulfilled + cancelled + pending + other
invoice_total = paid + outstanding + writeoff
```

Encode domain-specific conservation equations and use them as query validators.

### Strong domain

Finance, inventory, logistics, subscriptions, and funnel analytics.

---

## 3.10 Developmental / Morphogenetic Query Construction

Instead of a central planner writing the full query, let local semantic modules grow a plan using local constraints.

Each node sees nearby context and emits signals such as:

- `need_entity_key`
- `need_temporal_scope`
- `fanout_risk`
- `measure_not_additive`

A global relational structure emerges from repeated local updates.

This is speculative but tests whether complex plans can self-organize from modular constraints rather than one long reasoning trace.

---

# 4. Robotics, operating systems, networks, and reliability engineering

## 4.1 Schema-SLAM

SLAM jointly answers: `Where am I?` and `What does the world look like?`

A SQL agent faces an analogous problem:

- Where does the user's intent live in the schema?
- What is the semantic map of this unfamiliar database?

The agent incrementally builds a schema map while simultaneously localizing the task inside it.

### State

```text
belief over business region
map of discovered tables / edges / semantic landmarks
uncertainty over unexplored regions
```

### Research hypothesis

Joint map-building + localization beats one-shot schema retrieval on unfamiliar large databases.

---

## 4.2 Frontier-Based Schema Exploration

Mobile robots explore the boundary between known and unknown space. A schema agent can inspect the **frontier** around currently plausible tables.

Instead of globally searching every iteration:

```text
current region -> neighboring FK edges -> unexplored high-value frontier
```

This could be extremely efficient for schemas with thousands of tables but locally coherent domains.

---

## 4.3 Sensor-Fusion SQL

Treat metadata sources as noisy sensors:

- DDL
- foreign keys
- column statistics
- sample rows
- documentation
- BI dashboards
- historical queries
- data catalog tags

Assign each source a reliability model and fuse them probabilistically.

### Interesting setting

Documentation may be stale while query logs are current; FK metadata may be missing while values reveal joins.

The scientific problem becomes sensor fusion under conflicting evidence.

---

## 4.4 Schema Virtual Memory

A huge schema is larger than the LLM's working context, like RAM smaller than virtual address space.

Implement schema paging:

- pages = semantically coherent schema chunks
- page fault = reasoning requires unavailable metadata
- cache = currently loaded chunks
- eviction = learned replacement policy

Compare replacement strategies analogous to LRU, LFU, and learned caching.

### Metric

Task success per schema-token loaded.

---

## 4.5 Interrupt-Driven SQL Reasoning

Operating systems use interrupts for urgent events. A verifier should be able to preempt generation when a critical condition appears:

- destructive statement
- huge estimated scan
- Cartesian join
- access-control violation
- repeated contradiction

The architecture becomes event-driven rather than a rigid sequential chain.

---

## 4.6 Context Process Isolation

Multi-agent systems often share the same poisoned context and therefore hallucinate together.

Borrow process isolation:

- planner gets intent + schema summaries
- verifier gets intent + SQL + independent metadata
- safety monitor gets AST + policy only
- critic does not see writer rationale

### Hypothesis

Context isolation reduces correlated failure and post-hoc agreement compared with shared-context agents.

---

## 4.7 Routing-Protocol Join Discovery

Network routing protocols find paths under different assumptions. Model a schema graph similarly.

Each edge has multiple costs:

- semantic confidence
- cardinality risk
- historical usage
- latency / warehouse cost
- governance risk

A join path is chosen by routing over this multi-cost network, not simply shortest hop count.

Interesting variants:

- OSPF-like global link state
- BGP-like policy routing
- adaptive routes learned from successful analyst queries

---

## 4.8 Congestion-Control Reasoning Budget

Use TCP-like congestion control for inference budget.

Start cheap. If the agent sees clean evidence, increase slowly. If contradictions or verifier failures occur, back off and redirect budget to diagnosis.

Possible AIMD-style policy:

```text
successive clean checks -> additive increase in confidence / reduced verification
failure burst -> multiplicative increase in verification and stronger-model routing
```

The point is dynamic resource control, not a fixed number of reasoning calls.

---

## 4.9 SQL Chaos Engineering

Reliability should be tested under controlled infrastructure and metadata failures:

- stale docs
- missing FK metadata
- tool timeout
- one unavailable table
- changed column name
- corrupted sample row
- conflicting glossary entry
- rate-limited warehouse

### Benchmark metric

Graceful degradation curve rather than only clean-setting accuracy.

---

## 4.10 SRE Error Budgets for Semantic Reliability

Assign task classes an acceptable error budget.

Examples:

- exploratory dashboard query: 2% tolerated semantic risk
- finance close: near-zero tolerated risk
- destructive CRUD: explicit human approval required

The agent allocates verification, model strength, and human escalation according to the SLO.

### Research direction

Risk-adaptive agents rather than one universal architecture.

---

## 4.11 Circuit Breakers for Repair Loops

Repeated self-repair can consume cost while oscillating.

Detect patterns such as:

- same failure class repeats
- edit distance cycles between two plans
- verifier confidence does not improve

Open the circuit and switch strategy:

- ask user
- restart semantic interpretation
- call stronger model
- return abstention

Measure avoided wasted tokens and prevented unsafe persistence.

---

## 4.12 SagaSQL for Multi-Step Database Actions

Long database workflows resemble distributed transactions.

For agentic CRUD tasks, represent a plan as a saga:

```text
step A + compensation A'
step B + compensation B'
step C + compensation C'
```

If a later step fails, execute semantic compensations rather than assuming one monolithic transaction.

This is important for realistic tools where actions may span multiple systems.

---

# 5. Economics, markets, law, and accounting

## 5.1 Prediction-Market SQL

Give specialist agents a limited budget and let them stake it on explicit semantic claims:

- correct revenue source table
- correct join edge
- correct result grain
- need for clarification

A market mechanism aggregates beliefs.

### Why it might help

Ordinary majority voting treats all opinions equally. Markets force calibrated conviction and allow minority experts to dominate when they are repeatedly right.

### Evaluation

Calibration, accuracy, and resistance to one confidently wrong agent.

---

## 5.2 Token Auctions for Reasoning Resources

Modules bid for scarce inference budget.

Example bidders:

- schema explorer
- temporal specialist
- verifier
- candidate generator
- safety checker

Each predicts expected value of one more call. A controller allocates tokens to the highest marginal value.

This creates a concrete mechanism-design problem around agent compute.

---

## 5.3 Proper-Scoring Multi-Agent Incentives

Critics often learn to be verbose rather than useful. Give agents scoring rules that reward calibrated predictions of concrete failure events.

A critic must output:

```text
P(join_fanout_bug)=0.62
P(time_boundary_bug)=0.14
```

After verification, score with a proper scoring rule.

Over time, route weight toward critics that are actually calibrated.

---

## 5.4 Commit-Reveal SQL Deliberation

Herding is a major multi-agent problem. Borrow cryptographic / market commit-reveal:

1. each agent commits to a semantic plan hash;
2. all commitments are locked;
3. plans are revealed;
4. only then can agents critique one another.

### Hypothesis

Independent commitment increases useful disagreement and reduces correlated consensus errors.

---

## 5.5 Legal Burden of Proof for SQL

Different query classes require different evidence thresholds.

Example:

- ordinary analytics: `preponderance of evidence`
- financial reporting: `clear and convincing evidence`
- destructive write: `beyond reasonable doubt` + human confirmation

Translate those standards into required verifier passes and confidence thresholds.

### New research question

How should evidence standards change with action risk?

---

## 5.6 Case-Law Memory for Business Semantics

Enterprise definitions evolve through precedent:

- what counts as `active customer`
- which revenue table finance accepts
- how churn is defined

Store prior **adjudicated semantic cases**, including context and exceptions.

When a new task conflicts with precedent, surface the conflict instead of silently selecting the nearest example.

---

## 5.7 Argumentation-Graph Text2SQL

Represent competing interpretations as an argument graph.

Nodes:

- claims
- assumptions
- schema evidence
- user statements
- policy constraints

Edges:

- supports
- attacks
- undercuts

The final plan is the surviving argument under an explicit argumentation semantics.

This is qualitatively different from free-form debate because disagreement has typed logical structure.

---

## 5.8 Double-Entry Query Auditing

Accounting catches errors because every transformation is reconciled from two sides.

For suitable analytical queries, construct a second independent accounting view:

```text
revenue from order lines
vs.
revenue reconciled from invoice / ledger movements
```

The agent is trusted only when both sides reconcile within tolerance.

### High-value domains

Finance, inventory, payments, subscriptions, logistics.

---

## 5.9 Semantic Insurance Pricing

Estimate the expected loss of an incorrect answer:

```text
risk = P(error | evidence) * impact_if_wrong
```

Use this to determine whether to:

- answer immediately
- verify more
- ask user
- route to human analyst

This makes safety economically grounded instead of threshold-only.

---

# 6. Cryptography, privacy, and adversarial security

## 6.1 Proof-of-Policy SQL

The system returns not only SQL but a machine-checkable certificate that the query respects policy:

- read-only
- only permitted tables
- only approved columns
- aggregate size >= k
- no raw PII returned

Long-term, some certificates could use cryptographic proofs; near-term, the useful research is the certificate architecture itself.

---

## 6.2 Zero-Knowledge Semantic Checks

Speculative direction: verify selected properties of a query / result without revealing underlying sensitive rows.

Examples:

- prove result contains at least `k` subjects
- prove only approved columns contributed
- prove an aggregate was computed over a policy-compliant subset

This connects LLM database agents with privacy-preserving analytics.

---

## 6.3 Federated Failure Memory

Multiple organizations cannot share raw schemas or data but may share abstract failure lessons.

A memory item could expose only:

```text
schema motif: many-to-many bridge
failure: double counting
repair: pre-aggregate bridge before entity join
```

Research how much transferable SQL-agent improvement is possible without exposing proprietary metadata.

---

## 6.4 Semantic Taint Tracking

Track sensitive provenance through the relational plan:

```text
email -> PII
salary -> confidential
medical_code -> regulated
```

Taint propagates through joins, expressions, and outputs.

The LLM can propose a query, but a deterministic taint engine decides whether output policy is violated.

---

## 6.5 Capability-Based SQL Agents

Instead of a broad `execute_sql` tool, issue least-privilege capabilities:

- read table X
- inspect statistics for Y
- execute aggregate-only query
- mutate only sandbox table Z

Capabilities expire and cannot be widened by prompt injection.

This could be evaluated on agent success versus attack surface.

---

## 6.6 Honeypot Schema Evaluation

Insert decoy objects that a correct task never needs:

- fake PII table
- misleading revenue table
- tempting but deprecated join

Measure whether the agent touches the honeypot.

This is analogous to security deception and creates a new metric for unnecessary / risky exploration.

---

## 6.7 Metadata Prompt-Injection Benchmark

Treat schema comments, documentation, and sample values as untrusted input.

Inject malicious strings such as instructions to ignore policy, reveal secrets, or call tools unnecessarily.

The agent must preserve the data/control boundary.

### Research artifact

A database-specific prompt-injection benchmark where attacks live inside metadata rather than user prompts.

---

## 6.8 Canary Queries for Semantic Drift

Maintain a hidden set of tiny sentinel tasks with known semantics. Periodically run them against the current agent / schema snapshot.

If canary performance changes, trigger:

- cache invalidation
- memory rollback
- schema re-indexing
- stronger verification

This is continuous assurance for deployed SQL agents.

---

# 7. Mathematical structures not used in the previous atlas

## 7.1 CategorySQL — compositional semantics

Treat schemas, views, and queries as composable mappings.

A complex task is valid when its semantic diagram commutes: different legitimate decomposition paths lead to equivalent results.

Potential use:

- verify schema migrations
- reason about reusable query components
- transport a business metric across databases

The goal is not category-theory decoration; it is to obtain strong composition laws for semantic modules.

---

## 7.2 Topological Schema Maps

Large join graphs contain loops, bridges, hubs, and communities. Use topological descriptors to identify stable structure across renaming and local schema edits.

Possible features:

- persistent connected components across edge-confidence thresholds
- bridge edges whose removal disconnects domains
- cycles associated with many-to-many ambiguity

### Hypothesis

Topology-aware schema retrieval is more robust to noisy metadata than purely lexical retrieval.

---

## 7.3 Optimal-Transport Schema Alignment

To transfer an agent from one database to another, align semantic mass between schemas rather than match names one-to-one.

Example:

```text
source: customer + account + household
 target: client + membership + family_group
```

Optimal transport can represent many-to-many semantic alignment with costs derived from descriptions, values, and graph roles.

### Research task

Cross-company zero-shot query transfer.

---

## 7.4 Factor-Graph Text2SQL

Represent semantic choices as variables:

- entity mapping
- measure mapping
- join edge
- time interpretation
- result grain

Factors encode compatibility from schema, docs, values, and policy.

Then run approximate inference instead of one autoregressive reasoning chain.

### Benefit

Uncertainty is explicit and local; one piece of evidence can update all related choices.

---

## 7.5 Minimum-Description-Length SQL

When several plans fit the evidence, prefer the simplest semantic explanation:

```text
score = fit_to_intent - lambda * semantic_complexity
```

Complexity can penalize:

- unnecessary tables
- redundant joins
- gratuitous subqueries
- unexplained filters
- unsupported business assumptions

### Research question

Can MDL regularization reduce hallucinated joins and overcomplicated SQL?

---

## 7.6 Conformal SQL Abstention

Use conformal prediction / calibration ideas to provide a target coverage guarantee for selective answering.

The agent either:

- answers,
- asks,
- abstains.

The research target is not just average confidence but a controlled upper bound on error among accepted answers under a calibration regime.

This could be compelling for enterprise safety.

---

## 7.7 MaxSAT Semantic Planning

Let the LLM propose semantic clauses, then solve a constrained optimization problem.

Hard constraints:

- referenced columns exist
- permission rules
- key / type compatibility

Soft constraints:

- preferred business glossary mapping
- likely join path
- minimal table count
- historical analyst conventions

The MaxSAT solution becomes a semantic plan that optimally satisfies evidence and rules.

---

## 7.8 Information-Geometric Ambiguity

Represent competing interpretations as probability distributions over semantic plans. Define distances between them and measure whether a tool call meaningfully moves the belief state.

This yields a geometric definition of `how ambiguous` a task is and `how much` one observation resolved it.

Potential use: principled stopping criteria for schema exploration.

---

# 8. Linguistics, education, architecture, and organizational knowledge

## 8.1 Pragmatics-Aware SQL

Business questions depend on implicature and conversational expectations.

Example:

> "Which customers came back?"

The literal schema may support many definitions of `came back`, but conversational context may imply repeat purchase after a period of inactivity.

Model:

- literal semantics
- discourse context
- user role
- organizational conventions
- implicatures

### Benchmark

Create minimal pairs where literal wording is identical but prior conversational context changes the correct SQL.

---

## 8.2 Hermeneutic SQL Loop

Interpretation can alternate between the whole question and its parts:

1. tentative global meaning;
2. inspect each phrase under that meaning;
3. revise global meaning;
4. repeat until stable.

This is not ordinary iterative prompting if the system explicitly tracks which phrase changed interpretation and why.

---

## 8.3 Query Pattern Language

Architecture uses pattern languages to encode reusable solutions to recurring design problems.

Create a database-analytics pattern language:

- latest state per entity
- cohort retention
- funnel conversion
- inventory balance
- slowly changing dimension lookup
- bridge-table deduplication
- period-over-period comparison

The agent composes verified patterns rather than synthesizing every query from scratch.

### Research question

How much of enterprise analytics can be covered by a small verified pattern vocabulary?

---

## 8.4 Schema Urban Planning

Treat schema domains as city zones.

- local streets = common joins
- highways = shared dimensions
- dangerous intersections = many-to-many bridges
- zoning rules = governance boundaries

Cross-zone traversal requires stronger evidence than movement inside a well-understood local neighborhood.

The metaphor becomes useful if it yields a **hierarchical routing policy with governance costs**.

---

## 8.5 Analyst Apprenticeship Agent

Instead of training only from gold SQL, learn from how expert analysts investigate:

- which docs they open
- which sanity checks they run
- what questions they ask stakeholders
- how they validate result distributions

The target is a process model of expert inquiry rather than a mapping from question to final query.

---

## 8.6 User-Skill-Adaptive Clarification

A novice, analyst, and database engineer should receive different clarification questions.

Model a latent user skill state and optimize questions for:

- information gain
- cognitive burden
- terminology familiarity

This turns interactive Text2SQL into adaptive tutoring / collaboration rather than generic question asking.

---

# 9. Cross-field fusion systems

The most interesting systems may combine several mechanisms rather than borrow one metaphor in isolation.

## F1 — Schema-SLAM + Renormalization + Virtual Memory

A large-schema agent maintains a multi-resolution map, localizes intent, explores frontier regions, and pages only relevant schema chunks into working context.

**Target:** thousand-table enterprise databases.

---

## F2 — Predictive Coding + Sensor Fusion + Metacognitive Forecasting

Top-down intent predicts expected schema structure; DDL/docs/values act as noisy sensors; prediction errors update a calibrated failure forecast that selects the next tool.

**Target:** stale / incomplete metadata.

---

## F3 — Immune Danger Theory + SRE Error Budgets

Cheap negative-selection detectors continuously watch for semantic danger. Verification intensity escalates according to task risk and remaining error budget.

**Target:** low-cost production reliability.

---

## F4 — Retrosynthesis + Stoichiometric Balance + Double-Entry Audit

Generate the query backward from the desired result, enforce flow / balance constraints, and reconcile important measures through an independent accounting path.

**Target:** finance, payments, inventory, subscriptions.

---

## F5 — Prediction Market + Commit-Reveal + Proper Scoring

Specialists independently commit to semantic claims, reveal them, stake budget on confidence, and are rewarded based on verifier outcomes.

**Target:** multi-agent systems that avoid herding and verbosity.

---

## F6 — Dream Replay + Chaos Engineering + Honeypot Schemas

Offline, generate hard synthetic worlds with stale docs, missing metadata, decoy sensitive tables, and unusual join structures. Keep episodes that expose new failures and consolidate them into memory.

**Target:** continual robustness improvement.

---

## F7 — CategorySQL + Gauge Fixing + Semantic Symmetry Benchmark

Represent queries compositionally, canonicalize equivalent semantic descriptions, and verify invariance under schema transformations.

**Target:** cross-schema / cross-dialect transfer.

---

## F8 — Capability Security + Legal Burden of Proof + Conformal Abstention

Least-privilege tools limit what actions are possible; evidence requirements increase with risk; calibrated abstention protects cases that remain uncertain.

**Target:** enterprise action agents.

---

## F9 — Factor Graph + Quantum-Inspired Hypothesis Superposition + Active Observation

Maintain multiple incompatible semantic interpretations as a structured probabilistic state. Choose metadata / user questions that maximally separate them and collapse only after evidence.

**Target:** ambiguous business language.

---

## F10 — Ecological Agent Niches + Token Auction + Global Workspace

Specialized agents inhabit different schema/task niches, bid for compute only when relevant, and broadcast high-value findings through a small shared workspace.

**Target:** scalable multi-agent orchestration without context explosion.

---

## F11 — Cerebellar Residual Repair + Circuit Breaker + Case-Law Memory

Use a cheap residual model for common local fixes; if repairs oscillate, break the loop and retrieve adjudicated enterprise precedents or escalate.

**Target:** recurring production failures with strict cost limits.

---

## F12 — Topological Schema Map + Routing Protocol + Governance Costs

Find stable business communities and dangerous bridge edges topologically, then route join discovery using both semantic reliability and access-policy cost.

**Target:** complex governed enterprise warehouses.

---

# 10. New benchmark families suggested by these ideas

## B1 — Schema Navigation Benchmark

Measure how quickly an agent localizes a task in an unfamiliar giant schema.

Metrics:

- task success
- number of inspected objects
- path length to relevant schema region
- irrelevant exploration rate
- recovery after misleading landmark

---

## B2 — Semantic Symmetry Benchmark

Every task ships multiple equivalent schema / question representations. Score invariance / equivariance under known transformations.

---

## B3 — Metadata Reliability Benchmark

Systematically vary reliability of:

- docs
- FK metadata
- sample values
- glossary
- historical query logs

Measure whether the agent learns which source to trust.

---

## B4 — Semantic Risk Benchmark

Tasks carry impact levels. Score not only success but whether verification effort and abstention behavior are appropriate for risk.

---

## B5 — Agent Prompt-Injection-in-Data Benchmark

Place malicious natural language inside column comments, sample rows, documentation, or catalog metadata. Score policy integrity and unnecessary tool behavior.

---

## B6 — Continual Schema Drift Benchmark

A database evolves over episodes:

- column renamed
- table split
- new bridge introduced
- metric definition updated
- stale docs retained

Measure memory adaptation and catastrophic persistence of obsolete knowledge.

---

## B7 — Query Economics Benchmark

Give every task a token, latency, warehouse-scan, and human-interruption budget. Compare agent controllers on Pareto fronts rather than one accuracy number.

---

## B8 — Repair Oscillation Benchmark

Create tasks designed to cause repeated plausible but incompatible repairs. Evaluate circuit breaking, strategy switching, and graceful abstention.

---

# 11. Candidate paper-sized hypotheses

## W1 — Schema-SLAM

**Hypothesis:** joint schema mapping and intent localization improves success per inspected schema object on unfamiliar enterprise databases compared with static retrieval.

## W2 — Retrosynthesis SQL

**Hypothesis:** backward construction from the output contract reduces result-grain and double-counting errors on complex analytical tasks.

## W3 — PredictiveCodingSQL

**Hypothesis:** prediction-error-driven metadata acquisition uses fewer tool calls than flat retrieval while preserving or improving accuracy.

## W4 — Immune Danger Verification

**Hypothesis:** danger-triggered verifier escalation reaches near-exhaustive verification reliability at materially lower inference and database cost.

## W5 — Prediction-Market SQL

**Hypothesis:** market aggregation of independently committed semantic claims is better calibrated and more robust to one faulty agent than majority voting.

## W6 — Schema Virtual Memory

**Hypothesis:** learned schema paging outperforms static top-k context selection on schemas larger than the model context window.

## W7 — NoetherSQL

**Hypothesis:** symmetry-consistency training / selection reduces semantic brittleness under schema-preserving transformations.

## W8 — Conformal SQL Abstention

**Hypothesis:** calibrated selective answering can achieve a target accepted-answer error rate with substantially higher coverage than heuristic confidence thresholds.

## W9 — Topological Schema Routing

**Hypothesis:** topology-aware schema maps reduce dangerous cross-domain join errors on noisy or weakly documented databases.

## W10 — Dreaming SQL Agents

**Hypothesis:** verifier-filtered offline synthetic dreams improve robustness to unseen combinations of failure modes more than replaying original trajectories alone.

## W11 — Context Process Isolation

**Hypothesis:** isolating planner, critic, and safety contexts reduces correlated semantic failures compared with shared-context multi-agent systems under equal model-call budgets.

## W12 — Metadata Prompt-Injection Defense

**Hypothesis:** capability-based tools plus strict data/control separation sharply reduce attacks embedded in database metadata with small task-success loss.

---

# 12. Wildness / feasibility portfolio

| Idea | Novelty | Prototype difficulty | Scientific clarity | Best first setting |
|---|---:|---:|---:|---|
| Schema-SLAM | 5 | 3 | 5 | synthetic large schemas / Spider 2.0-like |
| Retrosynthesis SQL | 5 | 2 | 5 | analytical multi-join tasks |
| PredictiveCodingSQL | 5 | 3 | 4 | incomplete metadata |
| Immune danger verification | 5 | 2 | 5 | local benchmark + risk signals |
| Prediction-market SQL | 5 | 3 | 4 | structured multi-agent benchmark |
| Schema virtual memory | 4 | 3 | 5 | schemas beyond context window |
| NoetherSQL | 5 | 2 | 5 | transformation robustness suite |
| Conformal abstention | 4 | 3 | 5 | selective answering |
| Topological schema routing | 5 | 3 | 4 | large noisy schema graphs |
| Dreaming SQL agents | 5 | 4 | 4 | continual robustness |
| Process-isolated agents | 4 | 2 | 5 | correlated-failure ablation |
| Metadata prompt-injection defense | 5 | 3 | 5 | adversarial metadata suite |
| Zero-knowledge checks | 5 | 5 | 3 | privacy research |
| Morphogenetic query construction | 5 | 5 | 3 | synthetic program generation |
| CategorySQL | 5 | 5 | 3 | schema transformation / migration |

---

# 13. Recommended next wave for AutoResearchClaw

If the goal is to generate research that looks materially different from the current Text2SQL literature, a strong next portfolio is:

```text
Track A — Navigation
Schema-SLAM
+ multi-resolution schema map
+ virtual-memory paging

Track B — Reliability
Immune negative selection
+ danger-triggered verification
+ SRE error budgets

Track C — Semantic construction
Retrosynthesis SQL
+ balance constraints
+ independent reconciliation

Track D — Robustness
NoetherSQL symmetry benchmark
+ metadata chaos engineering
+ prompt-injection-in-data

Track E — Collective intelligence
Commit-reveal specialists
+ prediction market
+ token auction

Track F — Continual evolution
Hippocampal replay
+ verifier-filtered dreams
+ schema drift benchmark
```

These tracks are intentionally orthogonal. AutoResearchClaw could autonomously search within each family and then compare which mechanisms transfer across families.

---

# 14. A meta-idea: evolve the *scientific metaphors* themselves

Instead of only searching hyperparameters or agent graphs, let AutoResearchClaw search over imported mechanisms.

Example genome:

```yaml
representation: [retrosynthesis, factor_graph, gauge_fixed]
exploration: [slam_frontier, predictive_error, market_bid]
verification: [immune_detector, balance_check, symmetry_check]
memory: [replay, case_law, immune_memory]
controller: [error_budget, token_auction, basal_gate]
```

A research run can mutate one conceptual mechanism at a time, test held-out tasks, and retain only transfers that produce reproducible gains.

The long-term question is more interesting than `which prompt works best?`:

> **Which computational principles from other scientific fields survive transplantation into database reasoning, under controlled cost and robustness evaluation?**

That itself could become a methodology paper for autonomous cross-domain algorithm discovery.
