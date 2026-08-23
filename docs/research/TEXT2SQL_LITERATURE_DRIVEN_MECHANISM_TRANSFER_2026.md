# Text2SQL Literature-Driven Mechanism Transfer Study (2026)

> Date: 2026-08-23  
> Scope: literature-first ideation for Text2SQL / database agents.  
> Method: first map the 2025–2026 frontier, then import mechanisms from adjacent mature fields only when the transfer produces a falsifiable hypothesis and a concrete experiment.  
> Status: research spike, not an implementation specification.

---

## 1. Research protocol

This memo deliberately uses a stricter process than free-form analogy generation.

For each candidate direction:

1. identify an observed Text2SQL failure mode or an explicit gap in recent work;
2. find a mature mechanism in another field that addresses an analogous *structural* problem;
3. specify exactly what state, operator, objective, or feedback signal would be transferred;
4. check for overlap with recent Text2SQL work;
5. state a falsifiable hypothesis;
6. define baselines, metrics, and a minimal experiment;
7. reject ideas that are only metaphors or renamings of existing agent patterns.

The previous atlases already cover CEGIS, metamorphic verification, MCTS, typed relational IR, ordinary multi-agent debate, semantic self-consistency, schema-SLAM, MDL, prediction markets, causal intervention, abstract interpretation, process isolation, conformal abstention, and several other mechanisms. This memo therefore focuses on mechanisms that are materially different from those families.

---

# 2. Frontier audit: what strong Text2SQL systems are already doing

The current frontier is important because many seemingly novel ideas are already partially occupied.

## 2.1 Enterprise Text2SQL is now an agent problem

**Spider 2.0** moves far beyond Spider 1.0: 632 real-world enterprise workflow problems, often with >1,000 columns, Snowflake/BigQuery dialects, external knowledge and project context. Its paper reports that an o1-preview-based code agent solved only about 17% of tasks despite much stronger performance on earlier Text2SQL benchmarks.

Reference: Lei et al., *Spider 2.0: Evaluating Language Models on Real-World Enterprise Text-to-SQL Workflows*, 2024.  
https://arxiv.org/abs/2411.07763

**BIRD-Interact** makes interaction itself part of the task, with conversational and agentic database-interaction settings, hierarchical knowledge, a user simulator, and CRUD-style behavior. Reported success remains low even for strong contemporary models.

Reference: Huo et al., *BIRD-INTERACT: Reimagining Text-to-SQL Evaluation via Lens of Dynamic Interactions*, 2025.  
https://arxiv.org/abs/2510.05318

Implication: a proposal that merely adds ReAct, schema tools, or iterative execution is no longer differentiated.

## 2.2 Exploration and refinement are already strong baselines

**ReFoRCE** uses table compression, output-format restriction, iterative column exploration, self-refinement, parallel workflows, and voting for Spider 2.0.

Reference: Deng et al., *ReFoRCE: A Text-to-SQL Agent with Self-Refinement, Format Restriction and Column Exploration*, 2025.  
https://arxiv.org/abs/2502.00675

**FlexSQL** makes exploration and execution available throughout reasoning, generates multiple execution plans, can use SQL or Python, and backtracks from code-level errors to plan-level revisions.

Reference: Pham et al., *FlexSQL: Flexible Exploration and Execution Make Better Text-to-SQL Agents*, 2026.  
https://arxiv.org/abs/2605.02815

**AV-SQL** introduces agent-generated views / CTEs as intermediate structures for decomposing complex Text2SQL tasks.

Reference: Pham et al., *AV-SQL: Decomposing Complex Text-to-SQL Queries with Agentic Views*, 2026.  
https://arxiv.org/abs/2604.07041

Implication: “let the agent inspect columns, generate CTEs, execute, and revise” is an essential baseline, not a new contribution.

## 2.3 Memory and autonomous evolution are also occupied

**AgentSM** stores reusable structured semantic programs derived from prior traces and reports gains in both performance and trajectory efficiency on Spider 2.0 Lite.

Reference: Biswal et al., *AgentSM: Semantic Memory for Agentic Text-to-SQL*, 2026.  
https://arxiv.org/abs/2601.15709

**MIRA** (August 2026) decomposes historical corrections into reusable repair-memory items, verifies candidate memories against current database evidence, and adapts supported repair items to the current SQL. This directly raises the bar for any “failure memory” proposal.

Reference: Liu et al., *MIRA: Evidence-Verified Repair Memory for Text-to-SQL Correction*, 2026.  
https://arxiv.org/abs/2608.06950

**RoboPhD** already applies autonomous agent evolution to Text2SQL, evolving database-analysis and SQL-generation behavior over iterative experiments.

Reference: Borthwick & Ash, *RoboPhD: Self-Improving Text-to-SQL Through Autonomous Agent Evolution*, 2026.  
https://arxiv.org/abs/2601.01126

Implication for AutoResearchClaw: “autonomously evolve a Text2SQL agent” is no longer enough as a paper thesis. The research loop needs to discover or evaluate a more specific mechanism.

## 2.4 Robustness and business realism are becoming explicit problems

Kanchinadam et al. construct equivalent relational schemas from a shared conceptual model and show that Text2SQL behavior can change substantially even when underlying information needs remain equivalent. Exposing the conceptual E/R structure helps but does not remove the robustness problem.

Reference: Kanchinadam et al., *Same Data, Different Schemas: Robustness of LLM-based Text-to-SQL*, 2026.  
https://arxiv.org/abs/2605.25838

Business Logic-Driven Text-to-SQL synthesis argues that personas, scenarios, and workflows are necessary for realistic BI evaluation, with complex business queries remaining challenging.

Reference: Liu et al., *Business Logic-Driven Text-to-SQL Data Synthesis for Business Intelligence*, 2026.  
https://arxiv.org/abs/2601.14518

This suggests an underexplored frontier: **the hidden semantics of organizations, workflows, evolving beliefs, discourse, and dirty data**, not only SQL syntax or schema retrieval.

---

# 3. Idea 1 — ProcessModelSQL: use process mining as a semantic layer

## Source field

**Process mining**, especially object-centric process mining (OCPM).

Traditional process mining reconstructs process behavior from event logs. Object-centric process mining was developed because ERP/CRM data cannot always be represented faithfully as one case-id sequence: multiple interacting objects such as orders, invoices, shipments, customers, and payments participate in the same process.

Berti, Montali, and van der Aalst survey OCPM and explicitly motivate it by limitations of classical event logs for information systems with interacting business objects.

Reference: Berti et al., *Advancements and Challenges in Object-Centric Process Mining: A Systematic Literature Review*, 2023.  
https://arxiv.org/abs/2311.08795

There is also direct evidence connecting SQL and process-mining tasks, including declarative process queries expressed over relational event logs and recent Text2SQL-oriented process-mining datasets.

References:  
Schönig, *SQL Queries for Declarative Process Mining on Event Logs of Relational Databases*, 2015. https://arxiv.org/abs/1512.00196  
Yamate et al., *Text-to-SQL Oriented to the Process Mining Domain*, 2025. https://arxiv.org/abs/2509.09684

## Key transfer

Current Text2SQL normally treats a database primarily as:

```text
schema graph + descriptions + values
```

But many enterprise questions are really about a **business process**:

- orders approved but never shipped;
- customers who paid after cancellation;
- invoices reopened after settlement;
- opportunities that moved backward in a sales funnel;
- cases where payment occurred before required approval;
- time between successive lifecycle stages.

The proposed agent first mines or receives an **object-centric process model**, then uses that process model as a latent semantic layer between the natural-language question and SQL.

```text
NL question
   ↓
process interpretation
   - objects
   - activities
   - lifecycle state
   - ordering constraints
   - concurrency / handoffs
   ↓
object-centric process subgraph
   ↓
physical tables / event representations
   ↓
SQL
```

The process model can be a Petri-net-like model, directly-follows graph, Declare constraints, or a lighter object/activity graph.

## Why this is not ordinary schema RAG

A schema can tell us that `orders`, `payments`, and `shipments` exist. It usually does **not** tell us:

- which event logically precedes another;
- which transitions are normal or exceptional;
- which table encodes the current state versus event history;
- what constitutes one process instance when multiple objects interact.

Business Logic-Driven synthesis already uses workflows to generate realistic benchmark questions, but it does not establish runtime process-model reasoning as the semantic intermediate representation.

## Falsifiable hypothesis

> On workflow-centric enterprise questions, conditioning planning on an object-centric process model reduces temporal-order, lifecycle-state, and wrong-history-table errors compared with schema/value retrieval alone.

## Minimal experiment

Create 100–300 tasks over an ERP-like synthetic schema with orders, lines, invoices, shipments, payments, approvals, cancellations, and returns.

Conditions:

1. FlexSQL-like baseline with schema/value tools;
2. baseline + textual workflow docs;
3. baseline + automatically mined directly-follows graph;
4. baseline + object-centric process model;
5. oracle process model upper bound.

Metrics:

- execution/task success;
- temporal-order error rate;
- lifecycle-state error rate;
- history/current-state confusion rate;
- process objects inspected;
- token cost.

## Novelty / feasibility

Novelty: **high**, with adjacent process-mining/Text2SQL work but a different runtime mechanism.  
Feasibility: **medium-high** because mature process-mining tooling exists and synthetic ERP processes are easy to generate.

---

# 4. Idea 2 — TMS-SQL: truth-maintenance and belief revision for database agents

## Source field

**Belief revision**, truth-maintenance systems, and dynamic knowledge bases.

The AGM tradition studies how an agent should incorporate new information while making rational, often minimal, changes to prior beliefs. Truth-maintenance systems are an important computer-science precursor. Modern work continues to study belief-base and theory-base revision.

References:  
Stanford Encyclopedia of Philosophy, *Logic of Belief Revision*, substantive revision 2026. https://plato.stanford.edu/entries/logic-belief-revision/  
Fermé, Herzig & Martinez, *On the Logic of Theory Base Change*, AAAI 2025. https://ojs.aaai.org/index.php/AAAI/article/view/33636

The transfer is especially motivated by evidence that LLMs themselves are imperfect belief revisers: Belief-R studies whether models appropriately update prior reasoning in response to new evidence and reports substantial difficulty balancing revision with preservation of still-valid beliefs.

Reference: Wilie et al., *Belief Revision: The Adaptability of Large Language Models Reasoning*, 2024.  
https://arxiv.org/abs/2406.19764

## Key transfer

A database agent should maintain an explicit **justification graph**, not only a transcript.

Example belief state:

```yaml
beliefs:
  - id: B1
    claim: revenue_measure = invoices.net_amount
    confidence: 0.72
    sources: [catalog_doc_v3]
    status: tentative

  - id: B2
    claim: invoices.customer_id -> customers.id
    confidence: 0.98
    sources: [foreign_key_metadata]
    status: supported

  - id: B3
    claim: fiscal_quarter_starts_in_february
    confidence: 0.83
    sources: [finance_glossary_2025]
    status: supported
```

When new evidence conflicts — e.g. a newer finance document, a schema change, a contradictory query result, or a user correction — the agent does not simply append the new message or restart from scratch. It performs a **revision operator**:

```text
new evidence
   ↓
find dependent beliefs
   ↓
rank support / reliability / recency
   ↓
retract minimal unsupported set
   ↓
propagate consequences
   ↓
repair only affected relational decisions
```

## Difference from AgentSM / failure memory

AgentSM and MIRA are important baselines, but they primarily address reuse of structured traces or repair memories. TMS-SQL instead studies **rational contradiction handling and dependency-aware invalidation**.

This matters under:

- schema drift;
- stale documentation;
- conflicting metadata sources;
- business-definition changes;
- multi-turn user correction;
- cached semantic memory that has become obsolete.

## Falsifiable hypothesis

> Under controlled schema/documentation drift, justification-tracked minimal belief revision reduces stale-assumption persistence and repair regressions compared with append-only semantic memory and full re-planning.

## Benchmark proposal: EpistemicDB

Create episodes where the same logical task evolves:

```text
episode 1: revenue = invoice gross
episode 2: policy changes: refunds must be subtracted
episode 3: new table introduced
episode 4: old document remains retrievable
```

Measure:

- time-to-correct-update;
- stale belief survival rate;
- unnecessary belief churn;
- task success after update;
- number of unaffected decisions preserved;
- recovery after misleading evidence.

## Novelty / feasibility

Novelty: **very high**.  
Feasibility: **medium**; an initial version needs only an explicit belief graph and deterministic dependency invalidation, not a full theorem prover.

---

# 5. Idea 3 — DiscourseSQL: dynamic semantics instead of chat-history prompting

## Source field

**Discourse Representation Theory (DRT)** and dynamic semantics.

DRT was introduced specifically to model phenomena that sentence-at-a-time semantics handles poorly, including cross-sentence anaphora and tense. It maintains a discourse representation structure that is updated as discourse unfolds, with formal accessibility constraints controlling what earlier referents can be reused.

Reference: Stanford Encyclopedia of Philosophy, *Discourse Representation Theory*, updated 2024 / Spring 2026 edition.  
https://plato.stanford.edu/entries/discourse-representation-theory/

Conversational Text2SQL work has long observed that context-dependent turns are harder, and BIRD-Interact now makes multi-turn interaction central.

Reference: Parthasarathi et al., *Conversational Text-to-SQL: An Odyssey into State-of-the-Art and Challenges Ahead*, 2023.  
https://arxiv.org/abs/2302.11054

## Key transfer

Instead of giving the LLM the raw conversation and asking it to “understand context”, maintain a **Database Discourse Representation Structure**.

Example dialogue:

```text
U1: Show enterprise customers with more than $1M revenue last year.
U2: Only the ones in Europe.
U3: What about the previous quarter?
U4: Exclude those that churned before then.
```

The state explicitly contains:

```yaml
referents:
  population: enterprise_customers
  measure: revenue
  threshold: 1000000 USD
  region: Europe
  active_time_anchor: previous_quarter
  excluded_state: churned_before(active_time_anchor)

scope:
  population_accessible: true
  original_last_year_filter_superseded: true
  currency_assumption: inherited
```

Important operations include:

- introduce discourse referent;
- resolve anaphora / ellipsis;
- shift temporal anchor;
- supersede rather than conjoin an old filter;
- preserve accessible assumptions;
- model presuppositions explicitly.

## Why this is different from conversation summarization

A summary is untyped text. A DRT-inspired state has explicit semantics for **which prior entities, filters, and temporal anchors remain accessible**.

This directly targets errors such as:

- accidentally keeping an old time filter after “what about Q1?”;
- resolving “those customers” to the wrong population;
- treating “same regions” as a fresh unconstrained region set;
- failing to inherit a measure definition across turns;
- merging a correction with the old interpretation instead of replacing it.

## Falsifiable hypothesis

> A typed dynamic discourse state improves multi-turn SQL correctness specifically on anaphora, ellipsis, temporal-anchor shift, and correction turns, even when the raw dialogue is available to both methods.

## Experiment

Build targeted transformations over CoSQL/BIRD-Interact plus synthetic conversations.

Phenomenon labels:

- entity anaphora;
- set anaphora;
- ellipsis;
- temporal anaphora;
- correction / supersession;
- presupposition;
- scope shift.

Baselines:

1. raw chat history;
2. rolling natural-language summary;
3. generic key-value memory;
4. DRT-inspired typed discourse state.

Report per-phenomenon accuracy, not only aggregate success.

## Novelty / feasibility

Novelty: **high**; conversational Text2SQL exists, but formal dynamic-semantics-inspired state appears underexplored in the literature reviewed here.  
Feasibility: **high** for a constrained first version.

---

# 6. Idea 4 — SFL-SQL: semantic fault localization before repair

## Source field

**Software debugging**, particularly spectrum-based fault localization (SFL), program slicing, and delta debugging.

SFL ranks program elements by how strongly their execution correlates with failing versus passing tests. Program slicing narrows the statements that can influence an observed value. These methods separate **localization** from **repair**.

References:  
de Souza, Chaim & Kon, *Spectrum-based Software Fault Localization: A Survey of Techniques, Advances, and Challenges*, 2016. https://arxiv.org/abs/1607.04347  
Sasirekha et al., *Program slicing techniques and its applications*, 2011. https://arxiv.org/abs/1108.1352

Text2SQL repair is already active. A mutation-based repair method showed many failed predictions are close to correct; MapleRepair provides a detailed error taxonomy and argues that naive repair can incur high overhead and mis-repair; MIRA improves repair-memory reuse.

References:  
Yang et al., *On Repairing Natural Language to SQL Queries*, 2023. https://arxiv.org/abs/2310.03866  
Shen et al., *A Study of In-Context-Learning-Based Text-to-SQL Errors*, 2025. https://arxiv.org/abs/2501.09310  
Liu et al., *MIRA*, 2026. https://arxiv.org/abs/2608.06950

## Key transfer

Do not immediately ask an LLM to rewrite an incorrect SQL query.

First decompose the relational plan into semantic components:

```text
N1 tables
N2 join edge customer→orders
N3 join edge orders→refunds
N4 date filter
N5 status filter
N6 aggregation grain
N7 measure expression
N8 top-k
```

Run a collection of diagnostic tests / probes. Each probe touches or constrains a subset of nodes:

```text
T1 duplicate-grain test             covers N2,N3,N6   FAIL
T2 date-boundary test               covers N4         PASS
T3 refund conservation check        covers N3,N7      FAIL
T4 top-k stability check            covers N8         PASS
T5 schema/type check                covers N1,N2,N3   PASS
```

Compute suspiciousness scores analogous to Ochiai/Tarantula-style SFL and send only the high-suspicion semantic slice to the repair model.

## Stronger variant: SQL delta debugging

Given a long failing query/workflow, systematically remove joins, filters, CTE branches, or transformations while preserving the failure witness. Produce the **minimal failure-inducing relational slice**.

This is useful both for repair and for AutoResearchClaw failure analysis.

## Falsifiable hypothesis

> Explicit semantic fault localization reduces repair edit scope, token cost, and mis-repair of correct components compared with full-query self-refinement and memory-guided repair.

## Metrics

- final task success;
- localization top-1 / top-k accuracy using injected faults;
- fraction of correct semantic nodes modified;
- repair regression rate;
- tokens per successful repair;
- number of DB probes;
- edit distance at semantic-plan level.

## Minimal benchmark

Take gold queries, inject one or two controlled semantic faults:

- wrong join edge;
- missing DISTINCT;
- wrong temporal boundary;
- wrong aggregation;
- extra filter;
- wrong history table.

This creates ground-truth fault locations, which ordinary Text2SQL benchmarks lack.

## Novelty / feasibility

Novelty: **high but adjacent to repair literature**. The key differentiator is treating *localization as an independently evaluated problem*.  
Feasibility: **high**.

---

# 7. Idea 5 — ConstraintAcquisitionSQL: learn durable business rules by asking questions

## Source field

**Interactive constraint acquisition (CA)**.

Constraint acquisition attempts to learn a symbolic constraint model from examples or user answers. Recent work uses ML to guide which questions to ask and reports large reductions in the number of interaction queries required to converge.

Reference: Tsouros, Berden & Guns, *Learning to Learn in Interactive Constraint Acquisition*, 2023.  
https://arxiv.org/abs/2312.10795

The reported methods use classifiers to guide query generation, scope finding, and constraint identification, substantially reducing the interaction queries required in their evaluated settings.

## Key transfer

Current interactive Text2SQL usually asks clarification to solve **this one query**.

ConstraintAcquisitionSQL instead uses clarifications to learn **organization-level semantic contracts** that persist across future tasks.

Example hidden rules:

```text
active_customer(x) := last_completed_order(x) <= 90 days
                      AND NOT fraud_blocked(x)

net_revenue := captured_payment
               - settled_refund
               - chargeback

enterprise_account := account_tier IN {Strategic, Enterprise}
                      EXCEPT region = APAC where legacy_tier = 'E'
```

The agent maintains a candidate rule set and chooses high-information questions such as:

```text
"If a customer ordered 100 days ago but has an active annual subscription,
should they count as active?"
```

The answer updates a symbolic constraint model. Future queries inherit the learned rules.

## Difference from Ask-or-Act

Ask-or-Act optimizes whether a clarification is worthwhile for one task. Constraint acquisition optimizes **which query to ask so that the latent business-rule model converges across many tasks**.

It converts repeated enterprise Text2SQL from episodic QA into continual semantic model acquisition.

## Falsifiable hypothesis

> On repeated tasks from one organization, active constraint acquisition lowers cumulative human clarification cost while improving semantic accuracy more quickly than storing user corrections as examples or natural-language memory.

## Experiment

Create a simulated organization with 20–50 hidden business rules, including exceptions.

Over 500 sequential tasks compare:

1. no persistent learning;
2. example memory;
3. natural-language rule memory;
4. passive rule induction from corrections;
5. active constraint acquisition.

Metrics:

- cumulative task success;
- total user questions;
- rule precision/recall;
- semantic regret over time;
- transfer to unseen query templates;
- recovery after one rule changes.

## Novelty / feasibility

Novelty: **very high** relative to the Text2SQL literature reviewed here.  
Feasibility: **medium-high**, especially with a user simulator and synthetic hidden rules.

---

# 8. Idea 6 — CQA-Agent: Text2SQL when the database itself is inconsistent

## Source field

**Consistent Query Answering (CQA)** and database repairs.

CQA starts from a different premise than almost all Text2SQL work: the SQL may be perfectly correct while the **database violates integrity constraints**. An answer is considered consistent when it holds across all admissible repairs of the inconsistent database under the chosen repair semantics.

Foundational and systems references:

- Arenas, Bertossi & Chomicki, *Answer Sets for Consistent Query Answering in Inconsistent Databases*, 2002. https://arxiv.org/abs/cs/0207094
- Dixit & Kolaitis, *A SAT-based System for Consistent Query Answering*, 2019. https://arxiv.org/abs/1905.02828
- Staworko, Chomicki & Marcinkowski, *Prioritized Repairing and Consistent Query Answering in Relational Databases*, 2009. https://arxiv.org/abs/0908.0464

## Key transfer

Today Text2SQL evaluation usually assumes one database snapshot is authoritative. Real data often contains:

- duplicate supposedly unique IDs;
- contradictory customer states;
- orphan foreign keys;
- conflicting timestamps;
- multiple “current” records;
- inconsistent dimension mappings.

A CQA-aware agent distinguishes:

```text
certain answer   = true across all admissible repairs
possible answer  = true in at least one repair
unstable answer  = depends on conflict resolution
```

Example user question:

> How many active enterprise customers did we have at quarter end?

If three customer records have contradictory status histories, returning one precise number may be unjustified. The agent can instead produce:

```text
Certain count: 1,842
Possible range: 1,842–1,845
3 entities require conflict resolution
```

or ask the user which repair policy to apply.

## Why this is a different research axis

Most Text2SQL verification asks, “Did the generated SQL match the intent?”

CQA-Agent asks an orthogonal question:

> “Is the answer well-defined under imperfections of the underlying database?”

In the Text2SQL papers reviewed for this spike, CQA did not appear as a standard agent component; that makes this a promising novelty claim to investigate more rigorously before publication.

## Falsifiable hypothesis

> On databases with controlled integrity violations, repair-aware answering sharply reduces false certainty while preserving normal task accuracy on clean data.

## Benchmark: DirtyBIRD / RepairSpider

Inject constraint violations into existing DBs while retaining the same NL questions.

Publish:

- integrity constraints;
- conflict sets;
- repair semantics;
- certain / possible answer labels.

Metrics:

- certain-answer precision;
- false-certainty rate;
- coverage;
- user escalation rate;
- clean-data task success;
- computational overhead.

## Novelty / feasibility

Novelty: **very high**, subject to a dedicated novelty search before paper submission.  
Feasibility: **medium**; start with keys, FDs, and denial constraints where mature CQA algorithms exist.

---

# 9. Idea 7 — QueryArchaeology: mine the organization's latent semantic layer from SQL logs

## Source fields

**Database workload mining**, relational learning, and inductive logic programming.

Query-log research has long observed that analyst SQL contains knowledge absent from schemas. Wahl & Lenz describe SQL logs as a form of dynamic documentation containing expert knowledge about purpose, semantics, vocabulary, associations, and usage context of data sources.

Reference: Wahl & Lenz, *Analyzing SQL Query Logs using Multi-Relational Graphs*, 2017.  
https://ceur-ws.org/Vol-1917/paper01.pdf

Large-scale workload-mining systems also demonstrate that recurring query patterns can be mined from production SQL workloads.

Reference: Wang et al., *Real-time Workload Pattern Analysis for Large-scale Cloud Databases*, 2023.  
https://arxiv.org/abs/2307.02626

Inductive Logic Programming provides a mature framework for learning relational rules from examples plus background knowledge.

Reference: Lisi, *Learning Onto-Relational Rules with Inductive Logic Programming*, 2012.  
https://arxiv.org/abs/1210.2984

## Key transfer

AgentSM reuses structured prior traces. QueryArchaeology goes one abstraction level higher: infer a **latent semantic layer** from thousands or millions of historical human-written queries.

Mine patterns such as:

```text
canonical_join(orders, customers, orders.customer_id = customers.id)

finance_revenue_query -> usually excludes status IN ('void','test')

customer_snapshot -> selects max(valid_from) per customer as-of date

APAC_reporting -> converts local_currency using fx_daily on transaction_date
```

Important: these are not individual memories. They are **induced rules with support, exceptions, provenance, and temporal validity**.

The agent then reasons with:

```text
schema metadata
+ induced analyst conventions
+ query-frequency evidence
+ recency / team ownership
```

## Research questions

1. Can analyst query logs reveal semantic relationships that schema/FK metadata misses?
2. Do induced rules generalize better than retrieving nearest historical SQL examples?
3. Can temporal mining detect that a business definition changed?
4. Can team-specific conventions prevent applying Finance semantics to Marketing tasks?

## Falsifiable hypothesis

> A rule layer induced from historical analyst SQL improves held-out Text2SQL tasks on recurring enterprise schemas more than nearest-query retrieval at a lower inference-context cost.

## Experiment

Use a real or synthetic workload split chronologically.

Baselines:

- no query history;
- nearest SQL retrieval;
- AgentSM-style structured trace memory;
- query-template clustering;
- induced semantic-rule layer.

Metrics:

- held-out task success;
- context tokens;
- canonical join accuracy;
- business filter accuracy;
- rule precision/support;
- adaptation to temporal drift.

## Novelty / feasibility

Novelty: **high**; workload mining exists, and Text2SQL memory exists, but turning query logs into a learned declarative semantic layer is a different mechanism.  
Feasibility: **medium-high**.

---

# 10. Idea 8 — DefaultSQL: non-monotonic business semantics with explicit exceptions

## Source field

**Non-monotonic logic / default logic**.

Non-monotonic logic was built for defeasible reasoning: conclusions can be rationally retracted when exception information arrives. Reiter's default logic separates strict world facts from default rules whose conclusions hold only while their justifications remain consistent.

Reference: Stanford Encyclopedia of Philosophy, *Non-monotonic Logic*, updated 2024 / 2026 editions.  
https://plato.stanford.edu/entries/logic-nonmonotonic/

## Why this matters for enterprise SQL

Business definitions are often not clean universal rules.

Examples:

```text
Normally, an order counts as revenue when captured.
Exception: marketplace orders count at settlement.
Exception: internal/test accounts never count.

Normally, customer region comes from billing address.
Exception: strategic accounts use territory assignment.

Normally, active means activity in 90 days.
Exception: annual-contract customers remain active until contract end.
```

A flat glossary or one textual “definition” loses this structure.

## Key transfer

Represent semantic knowledge as:

```text
strict facts
+ defeasible defaults
+ exceptions
+ priority among rules
```

At query time, the system computes which defaults survive the current context and materializes them into the relational plan.

Example:

```text
DEFAULT active(c) IF recent_activity(c)
UNLESS annual_contract(c)

DEFAULT region(c)=billing_region(c)
UNLESS strategic_account(c)
THEN region(c)=territory_region(c)
```

## Difference from ordinary business-rule prompting

The contribution is not “put business rules in RAG”. It is **making exception structure executable and non-monotonic**, with an explicit consequence relation.

This can combine naturally with ConstraintAcquisitionSQL: interaction learns new defaults and exceptions.

## Falsifiable hypothesis

> Explicit default/exception reasoning reduces errors on exception-heavy business semantics compared with flat natural-language documentation and nearest-example retrieval, especially when a new exception invalidates a previously correct interpretation.

## Benchmark

Generate tasks from business rule sets with controlled exception depth:

```text
0 exceptions
1 exception
2 interacting exceptions
priority conflict
new exception introduced midstream
```

Measure task success and exception-specific accuracy.

## Novelty / feasibility

Novelty: **high** relative to the literature reviewed here.  
Feasibility: **medium-high** with a small Datalog/ASP/default-rule engine or a deterministic priority-rule implementation.

---

# 11. Three cross-mechanism systems worth testing

The following combinations have more scientific coherence than simply stacking agents.

## 11.1 ProcessTMS-SQL — a database agent that understands evolving workflows

Combine:

```text
object-centric process model
        +
truth-maintained semantic beliefs
        +
query generation
```

A schema change or workflow policy change invalidates only process beliefs that depend on it.

Research question:

> Can an agent preserve stable process knowledge while adapting rapidly to a changed workflow?

This is particularly suitable for enterprise systems where data models and operational processes co-evolve.

## 11.2 Discourse + Constraint Acquisition — conversations that teach the semantic model

Every clarification does two jobs:

1. resolve the current discourse referent / ambiguity;
2. decide whether the answer is a reusable business rule.

Example:

```text
User: “By active I mean any paying contract, even if they haven't logged in recently.”
```

The discourse layer resolves the present query; the constraint-acquisition layer proposes a durable rule update.

Research question:

> Can interactive Text2SQL amortize clarification cost by converting one-off corrections into validated reusable rules without overgeneralizing?

## 11.3 FaultLoc + CQA — distinguish query bugs from data inconsistency

A wrong result can have two fundamentally different causes:

```text
A. query semantics are wrong
B. query is right, database is inconsistent
```

Most LLM repair loops risk conflating these.

A joint diagnostic system first asks:

```text
Does the SQL violate intent tests?
Does the database violate assumed constraints?
Which explanation best accounts for the observed mismatch?
```

Then it either repairs SQL or reports unstable data.

Research question:

> Does explicit diagnosis of “query fault vs data fault” reduce destructive mis-repairs and false confidence?

This is a strong reliability paper because it changes the ontology of failure.

---

# 12. Recommended research portfolio

Ranking is based on differentiation from the current 2026 Text2SQL frontier, experimental clarity, and compatibility with AutoResearchClaw.

| Rank | Direction | Novelty | Feasibility | Experimental clarity | Why now |
|---|---|---:|---:|---:|---|
| 1 | **ConstraintAcquisitionSQL** | 5 | 4 | 5 | BIRD-Interact makes interaction central, but current clarification is mostly episodic |
| 2 | **TMS-SQL / BeliefRevisionSQL** | 5 | 3 | 5 | Agent memory is hot; rational contradiction handling remains a clear gap |
| 3 | **SFL-SQL semantic fault localization** | 4.5 | 5 | 5 | Repair is crowded, so explicit localization creates a clean new subproblem |
| 4 | **CQA-Agent / DirtyBIRD** | 5 | 3 | 5 | Moves evaluation from query correctness to answer epistemics under dirty data |
| 5 | **ProcessModelSQL** | 4.5 | 4 | 4.5 | Business-workflow realism is now measurable and enterprise benchmarks motivate it |
| 6 | **DiscourseSQL** | 4 | 5 | 5 | Interactive benchmarks expose context dependence; DRT gives a principled state model |
| 7 | **QueryArchaeology** | 4 | 4 | 4 | AgentSM/MIRA show memory value; rule induction is the next abstraction level |
| 8 | **DefaultSQL** | 4.5 | 4 | 4 | Explicit exceptions match real business semantics better than flat glossaries |

The scores are research-prioritization judgments, not literature-derived quantitative measurements.

---

# 13. What I would test first

## Experiment A — Semantic fault localization

Why first:

- requires no new massive benchmark;
- can inject known semantic faults into existing SQL;
- has objective localization ground truth;
- easy equal-budget comparison against self-refinement, rule-based repair, and MIRA-like repair memory;
- produces reusable instrumentation for AutoResearchClaw failure analysis.

Minimal thesis:

> **Locate before you repair: explicit semantic fault localization improves Text2SQL correction efficiency and reduces mis-repair.**

## Experiment B — Continual business-rule acquisition

Why second:

- highly differentiated;
- BIRD-Interact supplies the motivation for interaction;
- synthetic hidden-rule organizations make controlled experiments possible;
- connects naturally to AutoResearchClaw's self-evolution without duplicating RoboPhD.

Minimal thesis:

> **Clarifications should train the organization's semantic model, not disappear after one query.**

## Experiment C — Query fault vs data fault

Why third:

- classic Text2SQL assumes authoritative data;
- CQA gives strong database-theory foundations;
- benchmark construction is straightforward for key/FD violations;
- reliability value is easy to explain.

Minimal thesis:

> **A database agent should know whether it wrote the wrong query or queried a contradictory world.**

---

# 14. AutoResearchClaw integration as a research loop

These ideas can be researched without hard-coding them into the 23-stage core.

A useful autonomous loop is:

```text
Literature retrieval
      ↓
mechanism-transfer hypothesis
      ↓
novelty check against frontier methods
      ↓
benchmark / fault generator
      ↓
method prototype generated in sandbox
      ↓
trajectory + diagnostic logging
      ↓
paired evaluation under equal budget
      ↓
failure taxonomy update
      ↓
next mechanism / ablation proposal
```

The key change from generic autonomous agent evolution is that **the search space is a scientific mechanism with an explicit hypothesis**, not arbitrary code mutations.

For the eight directions in this memo, AutoResearchClaw can create new ARC-Bench-style research tasks such as:

- `SQL31`: Semantic fault localization before repair
- `SQL32`: Continual business constraint acquisition
- `SQL33`: Belief revision under conflicting database metadata
- `SQL34`: Object-centric process reasoning for ERP analytics
- `SQL35`: Dynamic discourse state for multi-turn SQL
- `SQL36`: Consistent query answering under key/FD violations
- `SQL37`: Semantic-rule induction from analyst query logs
- `SQL38`: Default/exception reasoning for business metrics

---

# 15. Literature map

## Current Text2SQL / database-agent frontier

- Lei et al. 2024 — Spider 2.0: https://arxiv.org/abs/2411.07763
- Huo et al. 2025 — BIRD-Interact: https://arxiv.org/abs/2510.05318
- Deng et al. 2025 — ReFoRCE: https://arxiv.org/abs/2502.00675
- Pourreza et al. 2024 — CHASE-SQL: https://arxiv.org/abs/2410.01943
- Talaei et al. 2024 — CHESS: https://arxiv.org/abs/2405.16755
- Pham et al. 2026 — FlexSQL: https://arxiv.org/abs/2605.02815
- Pham et al. 2026 — AV-SQL: https://arxiv.org/abs/2604.07041
- Biswal et al. 2026 — AgentSM: https://arxiv.org/abs/2601.15709
- Liu et al. 2026 — MIRA: https://arxiv.org/abs/2608.06950
- Borthwick & Ash 2026 — RoboPhD: https://arxiv.org/abs/2601.01126
- Kanchinadam et al. 2026 — Same Data, Different Schemas: https://arxiv.org/abs/2605.25838
- Liu et al. 2026 — Business Logic-Driven Text-to-SQL Data Synthesis: https://arxiv.org/abs/2601.14518
- Bhaskar et al. 2023 — AmbiQT / ambiguity: https://arxiv.org/abs/2310.13659
- Ding et al. 2025 — AmbiSQL: https://arxiv.org/abs/2508.15276

## Process mining

- Berti et al. 2023 — Object-centric process mining review: https://arxiv.org/abs/2311.08795
- Schönig 2015 — SQL for declarative process mining: https://arxiv.org/abs/1512.00196
- Berti & Qafari 2023 — LLMs for process mining: https://arxiv.org/abs/2307.12701
- Yamate et al. 2025 — Text2SQL for process mining: https://arxiv.org/abs/2509.09684

## Belief revision / dynamic knowledge

- SEP — Logic of Belief Revision: https://plato.stanford.edu/entries/logic-belief-revision/
- Fermé et al. 2025 — Theory Base Change: https://ojs.aaai.org/index.php/AAAI/article/view/33636
- Wilie et al. 2024 — Belief-R: https://arxiv.org/abs/2406.19764

## Dynamic discourse semantics

- SEP — Discourse Representation Theory: https://plato.stanford.edu/entries/discourse-representation-theory/
- Parthasarathi et al. 2023 — Conversational Text-to-SQL: https://arxiv.org/abs/2302.11054

## Program debugging / repair

- de Souza et al. 2016 — Spectrum-based fault localization survey: https://arxiv.org/abs/1607.04347
- Sasirekha et al. 2011 — Program slicing: https://arxiv.org/abs/1108.1352
- Yang et al. 2023 — Repairing NL-to-SQL: https://arxiv.org/abs/2310.03866
- Shen et al. 2025 — Text2SQL error study / MapleRepair: https://arxiv.org/abs/2501.09310

## Constraint acquisition / relational rule learning

- Tsouros et al. 2023 — Interactive Constraint Acquisition: https://arxiv.org/abs/2312.10795
- Lisi 2012 — Onto-relational ILP: https://arxiv.org/abs/1210.2984

## Inconsistent databases / CQA

- Arenas et al. 2002 — Answer Sets for CQA: https://arxiv.org/abs/cs/0207094
- Dixit & Kolaitis 2019 — CAvSAT: https://arxiv.org/abs/1905.02828
- Staworko et al. 2009 — Prioritized repairs: https://arxiv.org/abs/0908.0464

## Query-log mining

- Wahl & Lenz 2017 — SQL query logs as multi-relational graphs: https://ceur-ws.org/Vol-1917/paper01.pdf
- Wang et al. 2023 — Alibaba Workload Miner: https://arxiv.org/abs/2307.02626
- Sellam & Kersten 2017 — Mining database query logs: https://arxiv.org/abs/1703.08732

## Non-monotonic logic

- SEP — Non-monotonic Logic: https://plato.stanford.edu/entries/logic-nonmonotonic/

---

# 16. Bottom line

The literature scan suggests that the obvious 2024-era Text2SQL improvements — better schema retrieval, decomposition, execution feedback, multi-candidate generation, self-refinement, CTEs, memory, and even autonomous workflow evolution — are now well represented in the literature.

The more differentiated research opportunities move one level deeper:

1. **reason over business processes, not only schemas**;
2. **maintain and rationally revise semantic beliefs, not only append memory**;
3. **represent discourse state formally, not only replay chat history**;
4. **localize semantic faults before repairing SQL**;
5. **learn durable business constraints through interaction**;
6. **reason about uncertainty caused by inconsistent data itself**;
7. **induce organizational semantics from analyst behavior / query logs**;
8. **model business rules as defeasible defaults with explicit exceptions**.

These are not guaranteed wins. Their value is that each imports a mechanism with a mature theoretical or empirical foundation, produces a clear failure mode it should improve, and can be falsified with controlled experiments. That makes them better candidates for AutoResearchClaw than unconstrained architectural novelty.