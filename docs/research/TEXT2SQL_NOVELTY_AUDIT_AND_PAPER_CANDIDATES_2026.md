# Text2SQL Novelty Audit and Paper Candidate Refinement (2026)

> Date: 2026-08-23  
> Status: literature-driven research spike; no runtime implementation changes.  
> Goal: stress-test eight candidate directions against the closest prior work, reject overly occupied formulations, and retain only gaps that support a falsifiable paper contribution.

---

## 1. Audit protocol

For each candidate idea, this memo asks six questions:

1. **What is the closest prior work in Text2SQL / database agents?**
2. **What adjacent-field mechanism is being transferred?**
3. **Which part of the original idea is already occupied?**
4. **What exact gap survives the novelty check?**
5. **What is the smallest publishable contribution and killer experiment?**
6. **What is the most likely reviewer objection?**

The guiding rule is deliberately conservative: if a proposed contribution can be summarized as “another agent that detects, retrieves, asks, repairs, or remembers,” it is not considered sufficiently differentiated unless the representation, objective, supervision signal, or evaluation problem is materially new.

---

# 2. Candidate 1 — SpectrumSQL: semantic fault localization before repair

## Closest prior work

The original `SFL-SQL` idea is **not novel enough as originally phrased**.

Three recent lines already occupy much of generic “localize then repair”:

- **SHARE** (ACL 2025) explicitly targets precise error localization and granular correction. It transforms declarative SQL into stepwise action trajectories and performs hierarchical action correction.
- **ErrorLLM** (2026) explicitly models categorized Text2SQL errors with dedicated error tokens and shows that detection quality strongly determines refinement effectiveness.
- Recent fine-grained financial Text2SQL refinement work decomposes SQL into atomic units, executes them for localized feedback, and combines this with domain-rule grounding.

References:

- Qu et al., *SHARE: An SLM-based Hierarchical Action CorREction Assistant for Text-to-SQL*, ACL 2025. https://aclanthology.org/2025.acl-long.552/
- Hong et al., *ErrorLLM: Modeling SQL Errors for Text-to-SQL Refinement*, 2026. https://arxiv.org/abs/2603.03742
- Shen et al., *A Study of In-Context-Learning-Based Text-to-SQL Errors*, 2025. https://arxiv.org/abs/2501.09310

## Surviving gap

The differentiating idea is not “local error correction.” It is **spectrum-based semantic fault localization using independent executable tests**.

Borrow directly from software fault localization:

- program elements become semantic-plan nodes / SQL AST regions;
- test cases become targeted verifier probes;
- pass/fail spectra induce suspiciousness scores;
- repair is restricted to the highest-suspicion region.

Example plan nodes:

```text
N1 table selection
N2 customer-order join
N3 order-refund join
N4 date boundary
N5 status filter
N6 result grain
N7 measure expression
N8 top-k
```

Example test spectrum:

```text
                         N2 N3 N4 N6 N7 N8
unique-grain probe        x  x     x        FAIL
quarter-boundary probe          x           PASS
refund-balance probe         x        x     FAIL
top-k stability                              x PASS
schema validity            x  x              PASS
```

The key distinction from SHARE/ErrorLLM is that localization is derived from **behavioral evidence produced by a test suite**, not primarily a learned error classifier or a generated action trajectory.

## Minimal paper

**SpectrumSQL: Spectrum-Based Semantic Fault Localization for LLM-Generated SQL**

Build a benchmark by injecting one controlled semantic fault into otherwise-correct SQL:

- wrong join edge;
- missing pre-aggregation;
- `COUNT` vs `COUNT DISTINCT`;
- temporal off-by-one;
- wrong status filter;
- wrong latest-record logic;
- wrong measure source.

For every injected fault, preserve the exact ground-truth faulty node.

Compare:

1. whole-query LLM self-repair;
2. SHARE-style hierarchical correction if reproducible;
3. ErrorLLM-style categorical detection;
4. AST-diff / heuristic localization;
5. **verifier-spectrum suspiciousness + local repair**.

Metrics:

- top-1 / top-k localization accuracy;
- final repair success;
- fraction of correct nodes modified;
- repair regression rate;
- tokens per successful repair;
- DB probes per repair.

## Killer experiment

Show that, under equal repair budget, localization quality predicts repair success and that spectrum-guided repair changes fewer already-correct clauses.

## Reviewer objection

“SHARE already performs error localization.”

Required answer: the contribution must be framed as a **new supervision/evidence regime and benchmark**—fault localization from pass/fail semantic probes with ground-truth faulty nodes—not another hierarchical corrector.

## Verdict

**KEEP, but rename and sharply narrow.**  
Novelty: medium-high after refinement.  
Feasibility: high.  
Best first paper candidate: yes.

---

# 3. Candidate 2 — ConstraintAcquisitionSQL: learn organization semantics across tasks

## Closest prior work

Interactive disambiguation is now an active Text2SQL area:

- **Interactive Text-to-SQL via Expected Information Gain** (2025) maintains uncertainty over candidate SQLs and asks clarification questions that maximize expected information gain.
- **PRACTIQ** (NAACL 2025) benchmarks ambiguous and unanswerable conversational Text2SQL.
- **PleaSQLarify** (CHI 2026) operationalizes pragmatic repair through interactive clarification.
- BIRD-Interact includes multi-hop user/environment clarification flows.

References:

- Qiu et al., *Interactive Text-to-SQL via Expected Information Gain for Disambiguation*, 2025. https://arxiv.org/abs/2507.06467
- Dong et al., *PRACTIQ*, NAACL 2025. https://aclanthology.org/2025.naacl-long.13/
- PleaSQLarify, CHI 2026. https://sql-ambiguity.ivia.ch/

Adjacent-field mechanism:

- **Interactive Constraint Acquisition** learns an unknown symbolic constraint model by strategically querying a user. Tsouros et al. show ML-guided query selection can reduce the number of acquisition queries substantially.

Reference:

- Tsouros, Berden, Guns, *Learning to Learn in Interactive Constraint Acquisition*, 2023. https://arxiv.org/abs/2312.10795

## Occupied part

“Ask a clarification question when the current query is ambiguous” is already occupied.

## Surviving gap

Optimize clarification for **future cross-task semantic utility**, not only the current SQL.

A user answer can reveal an organization-level rule:

```text
annual-contract customers count as active until contract expiry,
even if no activity occurred in the last 90 days
```

The system should compile that answer into a reusable constraint and stop asking equivalent questions later.

Maintain a version space over candidate enterprise rules:

```yaml
concept: active_customer
candidate_rules:
  - recent_activity <= 90d
  - recent_activity <= 90d OR active_annual_contract
  - paid_invoice_within_180d
```

Question selection objective:

```text
current_task_gain
+ beta * expected_future_task_gain
- lambda * user_burden
```

This is qualitatively different from query-level EIG over current candidate SQLs.

## Minimal paper

**ConstraintAcquisitionSQL: Continual Learning of Business Semantics Through Clarification**

Construct a synthetic organization with 20–50 hidden reusable rules and exceptions. Generate a stream of 500 tasks where rules recur in different linguistic and SQL contexts.

Compare:

1. no clarification memory;
2. raw dialogue memory;
3. example retrieval;
4. passive rule extraction from answers;
5. query-level expected information gain;
6. **future-utility-aware active constraint acquisition**.

Metrics:

- cumulative execution accuracy;
- clarification turns per 100 tasks;
- rule precision / recall;
- future-task regret;
- transfer to unseen question forms;
- recovery after rule updates.

## Killer experiment

After 50 strategically selected clarification questions, the system should outperform a query-level clarification policy over the next 450 tasks while asking fewer additional questions.

## Reviewer objection

“The benchmark is synthetic and the rules are hand-designed.”

Mitigation: derive rule templates from enterprise BI patterns and historical SQL; validate a subset with real analysts; release the rule generator and streaming protocol.

## Verdict

**STRONG KEEP.**  
Novelty: high.  
Feasibility: medium.  
Long-term upside: very high.

---

# 4. Candidate 3 — TMS-SQL: dependency-aware belief revision for SQL-agent memory

## Closest prior work

Text2SQL memory is now occupied:

- **AgentSM** stores interpretable structured semantic memories derived from prior traces and reuses them to improve efficiency and accuracy.
- **MIRA** (Aug. 2026) decomposes historical SQL corrections into independently reusable repair-memory items and verifies their applicability against current DB evidence before adaptation.
- LinkedIn's enterprise Text2SQL system builds a knowledge graph from metadata, historical query logs, wikis, and code to capture up-to-date semantics.

References:

- Biswal et al., *AgentSM: Semantic Memory for Agentic Text-to-SQL*, 2026. https://arxiv.org/abs/2601.15709
- Liu et al., *MIRA: Evidence-Verified Repair Memory for Text-to-SQL Correction*, 2026. https://arxiv.org/abs/2608.06950
- Chen et al., *Text-to-SQL for Enterprise Data Analytics*, 2025. https://arxiv.org/abs/2507.14372

Adjacent-field mechanism:

- Doyle's **Truth Maintenance System** records justifications for beliefs and revises dependent beliefs when assumptions are contradicted.

Reference:

- Jon Doyle, *A Truth Maintenance System*, Artificial Intelligence 12(3), 1979.

## Occupied part

“Store prior semantic experience and retrieve it later” is not novel.

## Surviving gap

The missing operation is **retraction**.

Current memory methods mostly focus on:

```text
what to store
what to retrieve
when to activate
how to adapt
```

A TMS-based SQL agent focuses on:

```text
why is this belief held?
what depends on it?
what must be invalidated when evidence changes?
what unaffected beliefs should remain untouched?
```

Example:

```text
B1 revenue_source = invoices.net_amount
  evidence: finance_policy_2025

B2 orders.customer_id -> customers.id
  evidence: FK metadata

B3 report_metric_revenue depends_on B1
B4 query_template_Q17 depends_on B1, B2
```

When `finance_policy_2026` contradicts B1, the system invalidates B1, B3 and the affected part of B4, but preserves B2.

## Minimal paper

**TMS-SQL: Dependency-Directed Belief Revision for Continual Enterprise Text2SQL**

Create `EpistemicDB`, a sequential benchmark in which database semantics evolve:

- column renamed;
- canonical revenue source changes;
- glossary definition changes;
- new business exception added;
- old documentation remains accessible;
- one previous correction is later superseded.

Compare:

1. static RAG;
2. recency-weighted RAG;
3. AgentSM-style memory;
4. MIRA-style evidence-gated memory where applicable;
5. naive “overwrite latest fact” memory;
6. **dependency-aware truth maintenance**.

Metrics:

- stale-belief survival rate;
- unnecessary belief churn;
- time-to-correct-update;
- unaffected-knowledge preservation;
- downstream query accuracy after semantic change;
- explanation fidelity (`why is this rule active?`).

## Killer experiment

Inject a semantic change that affects exactly one metric family. TMS-SQL should update affected queries while preserving unrelated join and schema knowledge; global-memory refresh should either retain stale rules or unnecessarily damage unrelated behavior.

## Reviewer objection

“Is this just a knowledge graph with timestamps?”

Required distinction: timestamps say *when* evidence exists; a TMS explicitly represents **justifications and dependency-directed invalidation**.

## Verdict

**STRONG KEEP.**  
Novelty: high relative to current Text2SQL memory.  
Feasibility: medium-high.  
Particularly aligned with long-running enterprise agents.

---

# 5. Candidate 4 — CertaintySQL: distinguish query error from data inconsistency

## Closest prior work

Database theory has a mature field of **Consistent Query Answering (CQA)**. A database may violate integrity constraints; rather than arbitrarily trust one repair, a consistent answer is one that survives across the admissible repairs.

References:

- Arenas, Bertossi, Chomicki, *Consistent Query Answers in Inconsistent Databases*, foundational line of work.
- Dixit & Kolaitis, *A SAT-based System for Consistent Query Answering*, 2019. https://arxiv.org/abs/1905.02828
- Marconi & Rosati, *Consistent Query Answering for Existential Rules with Closed Predicates*, 2024. https://arxiv.org/abs/2401.05743

Text2SQL work increasingly acknowledges messy real data, but most evaluation still treats the benchmark DB instance as semantic ground truth.

## Surviving gap

Integrate **data uncertainty** into an LLM SQL agent and explicitly separate:

```text
query fault
from
data fault
```

A generated query can be semantically correct while the database contains:

- duplicated supposedly-unique entities;
- contradictory current-state rows;
- orphan keys;
- violated functional dependencies;
- conflicting status histories.

Instead of returning one overconfident answer, return:

```yaml
certain_answers: ...
possible_answers: ...
unstable_entities: ...
violated_constraints: ...
query_confidence: ...
data_consistency_warning: ...
```

## Minimal paper

**CertaintySQL: Text-to-SQL Under Inconsistent Databases**

Create a benchmark by injecting controlled integrity violations into otherwise valid Text2SQL databases:

- PK conflict;
- FD violation;
- orphan FK;
- multiple-current-row conflict;
- inconsistent temporal state;
- conflicting dimension mapping.

Evaluate two tasks:

1. produce the intended SQL;
2. decide whether the resulting answer is certain under the supplied integrity semantics.

Compare:

- ordinary Text2SQL;
- LLM verbal data-quality warning;
- heuristic constraint checks;
- single deterministic database repair;
- **CQA-based certain/possible answer layer**.

## Killer experiment

Show cases where a repair-focused SQL agent incorrectly changes a correct query because the true problem is dirty data. CertaintySQL should preserve the query and correctly attribute uncertainty to the database instance.

## Reviewer objection

“CQA is old database theory.”

That is precisely why the contribution must be the **new problem formulation and agent evaluation interface**, not a claim to invent CQA. The novelty is connecting executable LLM-generated analytics to principled inconsistency semantics and measuring false certainty.

## Verdict

**VERY STRONG KEEP.**  
Novelty: high as a Text2SQL evaluation/problem formulation.  
Database-theory grounding: excellent.  
Engineering difficulty: medium.

---

# 6. Candidate 5 — ProcSQL: object-centric business-process reasoning before SQL

## Closest prior work

Object-Centric Process Mining (OCPM) was introduced to address limitations of single-case process representations in ERP/CRM systems, where one event may involve orders, invoices, items, shipments, customers, and other object types simultaneously.

References:

- van der Aalst, *Object-Centric Process Mining: Unraveling the Fabric of Real Processes*, 2023.
- Berti, Montali, van der Aalst, *Advancements and Challenges in Object-Centric Process Mining: A Systematic Literature Review*, 2023. https://arxiv.org/abs/2311.08795

There is also a 2025 `text-2-SQL-4-PM` dataset for process-mining questions, but it primarily treats process-mining event-log queries as a domain-specific Text2SQL benchmark rather than using an object-centric process model as an intermediate reasoning representation.

Recent Text2SQL business-logic synthesis work models personas, scenarios and workflows to generate more realistic evaluation tasks, showing large residual performance gaps on complex business queries.

Reference:

- Liu et al., *Business Logic-Driven Text-to-SQL Data Synthesis for Business Intelligence*, 2026. https://arxiv.org/abs/2601.14518

## Occupied part

“Text2SQL for process-mining data” is already occupied.

“Use workflows to synthesize BI questions” is also occupied.

## Surviving gap

Use a **discovered or provided object-centric process model as the semantic planner for arbitrary enterprise relational data**.

Question:

> Which approved orders had still not shipped seven days later, excluding orders reopened after fraud review?

Schema-level reasoning sees tables and joins. Process-level reasoning sees:

```text
objects: order, approval, shipment, fraud_case
activities: created, approved, reopened, shipped
constraints:
  approved < shipped
  reopened invalidates prior terminal state
  seven-day duration measured from latest valid approval
```

Then map the process slice to physical relations.

## Minimal paper

**ProcSQL: Process-Model-Grounded Text-to-SQL for Enterprise Workflows**

Build tasks requiring event ordering, lifecycle states and interacting objects. Compare:

1. schema-only LLM;
2. RAG with process documentation as text;
3. generic graph-of-thought plan;
4. workflow summary;
5. **object-centric process subgraph -> SQL**.

Metrics:

- execution accuracy;
- temporal-ordering accuracy;
- lifecycle-state error rate;
- wrong-object / wrong-case correlation errors;
- transfer across schemas implementing the same process.

## Killer experiment

Use two physically different schemas implementing the same business process. A process-grounded model should transfer the semantic plan more reliably than schema-specific demonstrations.

## Reviewer objection

“Why not just retrieve the workflow documentation?”

The paper must show that a typed process representation gives benefits specifically on event ordering, object multiplicity, and lifecycle semantics that prose retrieval does not.

## Verdict

**KEEP.**  
Novelty: medium-high.  
Strongest use case: ERP / CRM / supply chain / finance workflow analytics.

---

# 7. Candidate 6 — DiscourseStateSQL: dynamic semantics for conversational Text2SQL

## Closest prior work

Conversational Text2SQL has a substantial history:

- SParC and CoSQL formalized context-dependent database dialogue.
- **HIE-SQL** uses both historical utterances and prior predicted SQL.
- **CQR-SQL** explicitly targets co-reference, ellipsis and user-focus change through conversational question reformulation.
- PRACTIQ, AmbiSQL, PleaSQLarify and BIRD-Interact expand ambiguity and clarification evaluation.

References:

- Zheng et al., *HIE-SQL*, 2022. https://arxiv.org/abs/2203.07376
- Xiao et al., *CQR-SQL*, 2022. https://arxiv.org/abs/2205.07686

Adjacent mechanism:

- **Dynamic Semantics / Discourse Representation Theory (DRT)** represents utterances as updates to an explicit discourse state and handles anaphora, scope, tense and presupposition.
- Modern DRT semantic parsing remains an active neurosymbolic problem; e.g. scope-enhanced compositional DRT parsing in 2024.

Reference:

- Yang et al., *Scope-enhanced Compositional Semantic Parsing for DRT*, 2024. https://arxiv.org/abs/2407.01899

## Occupied part

“Use conversational history to resolve co-reference and ellipsis” is old.

## Surviving gap

Use an **explicit update semantics** with first-class notions of:

- active discourse referents;
- inherited constraints;
- superseded constraints;
- temporal anchors;
- presuppositions;
- correction / retraction;
- local vs global scope.

Example:

```text
U1 Show enterprise customers above $1M last year.
U2 Only Europe.
U3 What about previous quarter?
U4 Exclude those already churned then.
```

State transition should explicitly record whether `previous quarter` replaces `last year`, what `those` refers to, and what time `then` anchors.

## Minimal paper

**DiscourseStateSQL: Explicit Dynamic Discourse State for Multi-Turn Database Querying**

Create transformation-controlled dialogues targeting:

- anaphora;
- ellipsis;
- temporal anchor shift;
- correction;
- constraint supersession;
- nested scope;
- presupposition failure.

Compare:

1. full raw dialogue;
2. generated conversation summary;
3. key-value memory;
4. question reformulation;
5. **typed discourse-state update**.

## Killer experiment

Adversarially append irrelevant or superseded historical turns. Explicit discourse-state systems should resist history-length and stale-context degradation better than raw-context methods.

## Reviewer objection

“This is CQR-SQL with a more complicated state.”

The paper needs a benchmark where **retraction, scope and supersession** are essential; simple anaphora/ellipsis tasks will not establish novelty.

## Verdict

**KEEP WITH CAUTION.**  
Novelty: medium if generic; high only with a strong dynamic-semantics benchmark.

---

# 8. Candidate 7 — Log2Rules: mine organizational semantics from historical SQL

## Closest prior work

Historical query logs are already being used:

- LinkedIn's enterprise system indexes historical query logs into a semantic knowledge graph.
- **HI-SQL** generates contextual hints from historical query logs.
- **Query-Log-Informed Schema Descriptions** mines logs to automatically improve schema documentation for Text2SQL.
- BEAVER itself is sourced from real enterprise query logs.

References:

- Chen et al., 2025. https://arxiv.org/abs/2507.14372
- Parab et al., *HI-SQL*, 2025. https://arxiv.org/abs/2506.18916
- Egorova, *Query-Log-Informed Schema Descriptions and their Impact on Text-to-SQL*, 2025.
- Chen et al., *BEAVER: An Enterprise Benchmark for Text-to-SQL*, 2024. https://arxiv.org/abs/2409.02038

## Occupied part

“Retrieve similar historical SQL” and “use logs to enrich schema descriptions” are occupied.

## Surviving gap

Perform **cross-query rule induction**, not retrieval.

From thousands of analyst queries, induce reusable organizational invariants such as:

```text
finance revenue queries usually exclude test and void transactions
quarter-end customer status uses latest valid_from before report date
APAC sales queries convert currency at transaction-date FX
subscription churn queries ignore grace-period states
```

Each induced rule should carry:

```yaml
support: 0.91
counterexamples: 23
team_scope: finance
valid_from: 2025-02-01
valid_to: null
provenance_queries: [...]
confidence: ...
```

The goal is to convert a workload into a **versioned semantic rule layer**.

## Minimal paper

**Log2Rules: Mining Enterprise SQL Workloads into Reusable Business Semantics**

Take historical query corpora and hide selected queries/tasks. Compare:

1. nearest-query retrieval;
2. query-log schema descriptions;
3. cluster summaries;
4. LLM-generated rules without validation;
5. **validated rule induction + rule-grounded Text2SQL**.

Rule validation can use held-out query consistency and DB execution probes.

Metrics:

- downstream Text2SQL accuracy;
- rule precision;
- rule support / exception coverage;
- temporal rule drift detection;
- cross-team negative transfer;
- token savings vs raw log retrieval.

## Killer experiment

Show that one induced rule improves many semantically related but lexically dissimilar tasks where nearest-neighbor query retrieval fails.

## Reviewer objection

“This is just workload mining / documentation generation.”

The system must produce explicit, executable or machine-checkable **semantic rules with exceptions and validity intervals**, not prose documentation.

## Verdict

**KEEP AFTER REFRAMING.**  
Original QueryArchaeology formulation: too close to existing log use.  
Log2Rules formulation: substantially stronger.

---

# 9. Candidate 8 — DefeasibleSQL: business rules with defaults and exceptions

## Closest prior work

Text2SQL systems increasingly use business-rule context, and recent financial refinement work grounds correction in a domain rule base. Business-logic synthesis also explicitly models personas, workflows and KPI logic.

However, these systems generally present rules as retrieved text or static context.

Adjacent mechanism:

- **Non-monotonic reasoning / Default Logic** provides a formal semantics for conclusions that hold normally but may be defeated by exceptions.

Example:

```text
DEFAULT:
  customer_region = billing_region

UNLESS:
  strategic_account

THEN:
  customer_region = territory_region
```

Reference:

- Reiter's default logic and modern overviews of non-monotonic logic; see Stanford Encyclopedia of Philosophy, *Non-monotonic Logic*.

## Surviving gap

Enterprise analytics semantics are often **defeasible rather than universal**:

```text
normally active = activity in last 90d
except active annual-contract customers
except suspended compliance accounts
```

A retrieval-only LLM must resolve priority and exception interactions implicitly. DefeasibleSQL would make them first-class and compute which defaults survive in the current context before SQL synthesis.

## Minimal paper

**DefeasibleSQL: Exception-Aware Business Semantics for Text-to-SQL**

Build a benchmark with rule hierarchies:

- base definition;
- exception;
- exception-to-exception;
- context-specific override;
- temporal override;
- conflicting department policy.

Compare:

1. prose rules in prompt;
2. retrieved rule chunks;
3. generated chain-of-thought;
4. ordinary rule engine with hard rules only;
5. **default/exception inference -> resolved semantic contract -> SQL**.

Metrics:

- execution accuracy;
- exception handling accuracy;
- rule-priority accuracy;
- consistency under irrelevant-rule injection;
- behavior after adding a new exception.

## Killer experiment

Add one new exception after deployment. A defeasible system should retract only the default-derived conclusions it defeats, while a plain prompt-memory system should show higher inconsistency or stale behavior.

## Reviewer objection

“Business rules can just be encoded as ordinary if/else logic.”

Response: the benchmark must contain incomplete information, competing defaults and rule priority where monotonic hard-rule encoding is either brittle or requires manually enumerating all exception combinations.

## Verdict

**KEEP AS HIGH-RISK / HIGH-REWARD.**  
Novelty: high.  
Benchmark design difficulty: high.  
Best paired with ConstraintAcquisitionSQL or TMS-SQL.

---

# 10. Audit summary

| Candidate | Original novelty after audit | Refined direction | Verdict |
|---|---:|---|---|
| SFL-SQL | low-medium | **SpectrumSQL**: test-spectrum semantic fault localization | keep, narrow |
| ConstraintAcquisitionSQL | high | persistent cross-task rule acquisition | strong keep |
| TMS-SQL | high | dependency-aware belief retraction/revision | strong keep |
| CQA-Agent | high | **CertaintySQL**: query-vs-data uncertainty | very strong keep |
| ProcessModelSQL | medium-high | **ProcSQL**: object-centric process-plan grounding | keep |
| DiscourseSQL | medium | **DiscourseStateSQL** with scope/retraction/supersession | keep with caution |
| QueryArchaeology | low-medium | **Log2Rules**: cross-query semantic rule induction | keep after reframing |
| DefaultSQL | high | **DefeasibleSQL**: defaults + exceptions + priorities | high-risk keep |

---

# 11. Revised paper portfolio

## Tier 1 — cleanest near-term papers

### P1. SpectrumSQL

Why first:

- controlled fault injection gives ground truth;
- evaluation is objective;
- implementation can start on BIRD/Spider-derived SQL;
- reviewer comparison set is clear (SHARE, ErrorLLM, MapleRepair-style repair).

Core thesis:

> Semantic repair improves when the system localizes the faulty relational decision from independent executable tests before editing.

### P2. CertaintySQL

Why first:

- introduces a genuinely different evaluation axis;
- grounded in mature database theory;
- can create a benchmark without training a new LLM;
- directly attacks false certainty on messy enterprise data.

Core thesis:

> A Text2SQL system should not repair a correct query to compensate for inconsistent data; it should separate query uncertainty from data uncertainty.

## Tier 2 — larger but potentially more important

### P3. TMS-SQL / EpistemicDB

Core thesis:

> Long-running database agents need principled retraction and dependency-aware semantic revision, not only retrieval memory.

### P4. ConstraintAcquisitionSQL

Core thesis:

> Clarification should optimize cumulative organizational learning, not only the current query.

## Tier 3 — differentiating enterprise semantics

### P5. ProcSQL

Process semantics as a portable intermediate layer across ERP schemas.

### P6. Log2Rules

Mine query workloads into validated, versioned semantic rules rather than retrieve examples.

### P7. DefeasibleSQL

Treat business definitions as defaults with exceptions and priorities.

### P8. DiscourseStateSQL

Use explicit update semantics for conversational corrections, temporal anchors and superseded constraints.

---

# 12. Strong fusion hypotheses that survive the audit

The audit suggests several combinations that are more coherent than a generic multi-agent stack.

## Fusion A — Organizational Semantics Compiler

```text
Historical SQL logs ──> Log2Rules ──┐
                                    │
User clarification ─> Constraint Acquisition
                                    │
                                    v
                         Defeasible Rule Base
                                    │
                                    v
                           TMS / Belief Revision
                                    │
                                    v
                        resolved semantic contract
                                    │
                                    v
                                  SQL
```

Scientific question:

> Can an enterprise SQL agent learn, revise and execute organizational semantics as an explicit symbolic layer?

This is much more differentiated than “RAG over enterprise documentation.”

## Fusion B — Diagnosis Before Repair

```text
Generated SQL
    ↓
Verifier probe suite
    ↓
SpectrumSQL localization
    ↓
        Is the failing evidence caused by query logic?
        /                                      \
      yes                                      no / ambiguous
      ↓                                             ↓
local SQL repair                              CertaintySQL / data diagnosis
```

Scientific question:

> Can we prevent mis-repair by first distinguishing query faults from data faults?

This is an especially clean research program.

## Fusion C — Process + Discourse Semantics

```text
Dialogue state
   ↓
DiscourseStateSQL
   ↓
current intent + temporal anchors
   ↓
Object-centric process model
   ↓
process slice
   ↓
physical schema mapping
   ↓
SQL
```

This directly targets long-running enterprise BI conversations whose questions are both context-dependent and workflow-dependent.

---

# 13. Recommended next research cycle

The next spike should not brainstorm more names. It should build an **evidence table for the top four candidates**:

1. SpectrumSQL
2. CertaintySQL
3. TMS-SQL
4. ConstraintAcquisitionSQL

For each candidate, search and record:

- 10–20 closest papers;
- exact task / datasets used;
- whether code is public;
- what supervision they require;
- what signals they use at inference time;
- whether they measure regressions / false corrections;
- whether they model sequential semantic change;
- whether they distinguish data uncertainty from query uncertainty;
- cost metrics;
- strongest reviewer overlap risk.

Then convert the evidence into one-page pre-registration style experiment cards:

```text
claim
minimum method
primary metric
primary baseline
falsification condition
compute budget
expected failure mode
```

Only after that should one candidate graduate from a research spike into an implementation design.

---

# 14. References / frontier anchors

- Spider 2.0: https://arxiv.org/abs/2411.07763
- BIRD-Interact: https://arxiv.org/abs/2510.05318
- FlexSQL: https://arxiv.org/abs/2605.02815
- AV-SQL: https://arxiv.org/abs/2604.07041
- AgentSM: https://arxiv.org/abs/2601.15709
- MIRA: https://arxiv.org/abs/2608.06950
- RoboPhD: https://arxiv.org/abs/2601.01126
- SHARE: https://aclanthology.org/2025.acl-long.552/
- ErrorLLM: https://arxiv.org/abs/2603.03742
- PRACTIQ: https://aclanthology.org/2025.naacl-long.13/
- Interactive Text-to-SQL via EIG: https://arxiv.org/abs/2507.06467
- Interactive Constraint Acquisition: https://arxiv.org/abs/2312.10795
- CAvSAT / CQA: https://arxiv.org/abs/1905.02828
- OCPM survey: https://arxiv.org/abs/2311.08795
- Business Logic-Driven Text-to-SQL Synthesis: https://arxiv.org/abs/2601.14518
- Enterprise Text-to-SQL: https://arxiv.org/abs/2507.14372
- HI-SQL: https://arxiv.org/abs/2506.18916
- BEAVER: https://arxiv.org/abs/2409.02038
- HIE-SQL: https://arxiv.org/abs/2203.07376
- CQR-SQL: https://arxiv.org/abs/2205.07686
- DRT semantic parsing: https://arxiv.org/abs/2407.01899
