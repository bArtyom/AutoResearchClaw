# CertaintySQL Phase-0 Experiment Handoff (2026)

> Date: 2026-08-23  
> Status: research handoff; stop point before compute-heavy implementation/experiments.  
> Purpose: preserve the current research state and define the first machine-executable experiment for `Fault-or-Data?` / `CertaintySQL`.

---

## 1. Why stop research-chat here

The literature/novelty work has progressed far enough that the next useful step is no longer more ideation. The highest-value next action is a controlled pilot on real code, databases, and model runs.

The core hypothesis is narrow and falsifiable:

> **When SQL is semantically correct but the database violates integrity assumptions, current Text2SQL/self-repair agents may incorrectly modify the correct SQL instead of diagnosing a data fault.**

If this phenomenon is weak, the broader CertaintySQL direction should be downgraded. If it is strong, it justifies investment in query-vs-data diagnosis, formal data evidence, and consistent/range-consistent query answering.

---

## 2. Current research conclusion

The novelty claim must **not** be “LLM + Consistent Query Answering.” CQA, database repairs, SAT/MaxSAT encodings, tractable rewritings, and aggregation-aware/range-consistent answers are mature research topics.

The defensible contribution is the diagnostic problem:

```text
Given:
- natural-language intent
- schema
- candidate SQL
- database instance
- integrity constraints / evidence

diagnose one of:
- OK
- QUERY_FAULT
- DATA_FAULT
- BOTH
- INSUFFICIENT_EVIDENCE
```

and then choose the correct intervention layer:

```text
QUERY_FAULT -> repair SQL
DATA_FAULT  -> preserve SQL; report/quantify data uncertainty
BOTH        -> repair SQL and report data uncertainty
```

`CertaintySQL` is best treated as the answer-certification/evidence component inside this larger `Fault-or-Data?` task.

---

## 3. Phase-0 research question

Before building CQA infrastructure, test whether the motivating failure actually occurs.

### Primary question

When shown suspicious execution evidence from a corrupted database while the SQL is still correct, how often does a strong SQL repair agent rewrite the correct SQL?

### Primary metric

```text
Misrepair Rate =
  # DATA_FAULT-only episodes where correct SQL is modified
  ---------------------------------------------------------
  # all DATA_FAULT-only episodes
```

### Important secondary metrics

- semantic regression rate after repair
- final task success
- unnecessary SQL AST edit rate
- edit locality by SQL clause / semantic role
- token cost
- tool-call count
- latency
- abstention / escalation rate
- false diagnosis distribution

---

## 4. Go / no-go criterion

Pre-register a conservative stopping rule.

```text
If representative strong repair baselines show
Misrepair Rate < ~5%
under controlled DATA_FAULT-only episodes,
then the central phenomenon is too weak to justify
building the full CertaintySQL system in its current form.
```

If misrepair is materially above that level and reproducible across models / fault types, proceed to Phase-1.

The exact inferential threshold should be reported with confidence intervals rather than treated as a magical hard cutoff, but the ~5% rule is the operational go/no-go prior.

---

## 5. Controlled 2 x 2 benchmark design

Start from a clean task:

```text
Q   = natural-language question
D*  = clean database
S*  = semantically correct SQL
```

Construct four controlled conditions:

| Database | SQL | Ground-truth label |
|---|---|---|
| clean | correct | `OK` |
| clean | mutated/faulty | `QUERY_FAULT` |
| corrupted | correct | `DATA_FAULT` |
| corrupted | mutated/faulty | `BOTH` |

Phase-0 should focus primarily on `DATA_FAULT`, because that is the cleanest test of misrepair.

The benchmark must preserve exact provenance for every injected corruption and SQL mutation.

---

## 6. Phase-0 data-fault families

Use only faults with explicit, machine-checkable semantics first.

Recommended initial families:

1. **PK / UNIQUE conflict**
   - duplicate business identifier
   - duplicate current-state record

2. **Orphan foreign key**
   - child references missing parent

3. **Simple functional-dependency violation**
   - one determinant maps to conflicting dependent values

4. **Multiple-current-row / temporal consistency fault**
   - two rows simultaneously claim `is_current = true`
   - overlapping validity intervals where only one state should be valid

5. **Conflicting categorical state**
   - same entity simultaneously has incompatible statuses

6. **Duplicate business event**
   - duplicate payment/refund/order-event row that creates plausible but wrong aggregate behavior

Do **not** start with arbitrary numeric corruption, free-text inconsistency, complex missing-not-at-random corruption, multi-system reconciliation, or uncertain business-rule discovery. Those make the repair semantics ambiguous and confound the core experiment.

---

## 7. SQL-fault families for the full 2 x 2 benchmark

For `QUERY_FAULT` and `BOTH`, use controlled semantic mutations such as:

- wrong join edge / join key
- missing predicate
- extra predicate
- boundary shift (`<` vs `<=`, date-window shift)
- `COUNT` vs `COUNT DISTINCT`
- missing pre-aggregation before fan-out join
- wrong aggregation function
- wrong grouping grain
- current-state table vs history-table mistake
- inner vs left join where semantics differ

Each mutation must be independently validated to be incorrect for the task.

---

## 8. Benchmark invariants

Before model runs, tests must prove:

1. clean `S*` is correct on the clean database;
2. the data corruption does not turn `S*` into a query fault;
3. the corruption violates exactly the intended constraint/fault family where possible;
4. the injected SQL mutation is incorrect independently of the data corruption;
5. labels are derivable from construction metadata, not model judgement;
6. no trivial lexical field reveals the class label;
7. random seeds and snapshots reproduce the same episode exactly.

The single most important invariant is:

> In `DATA_FAULT`, SQL must remain semantically correct even when the observed result is surprising or uncertain.

---

## 9. Phase-0 baselines

At minimum compare:

### B0 — No repair

Keep the SQL unchanged. This establishes how often repair is actually necessary.

### B1 — Generic LLM repair

Give the model intent, SQL, and suspicious result/error evidence and ask it to fix the query if needed.

### B2 — Execution-feedback repair

Allow execution/result evidence and iterative SQL revision.

### B3 — Strongest available repair/self-correction baseline

Use the strongest realistic baseline that can be integrated under the same model/tool budget.

Do not implement the full CertaintySQL/CQA system before the Phase-0 phenomenon test.

---

## 10. Experimental controls

Freeze and record:

- model name/version
- system prompt
- repair prompt
- temperature / sampling parameters
- maximum tokens
- number of repair rounds
- tool budget
- DB engine version
- benchmark seed
- corruption seed
- SQL mutation seed
- database snapshot hash
- full trajectory

Use paired evaluation whenever possible: the same task should appear in clean and corrupted variants so that the effect of data inconsistency is isolated.

---

## 11. Failure taxonomy to log

For every episode, record whether the agent:

- correctly keeps SQL unchanged
- correctly diagnoses data fault
- modifies irrelevant SQL region
- adds `DISTINCT` to hide duplicates
- changes join to suppress conflicting rows
- drops rows via stronger filtering
- chooses arbitrary `MAX`/`MIN` to collapse inconsistent state
- changes aggregation to force plausible totals
- globally rewrites a locally correct query
- abstains / escalates
- recognizes both query and data fault

Particularly valuable are **plausible cover-up repairs**: changes that make the output look cleaner while hiding the underlying data problem.

---

## 12. Required Phase-0 artifacts

The machine experiment should produce:

```text
benchmark generator
constraint/fault definitions
SQL mutation library
unit tests / property tests
benchmark manifest
clean and corrupted DB snapshots
raw model trajectories
structured episode logs
analysis script
result table
confidence intervals
failure taxonomy
representative misrepair cases
go/no-go memo
```

Do not report only aggregate accuracy. Preserve enough evidence to audit why the agent changed a query.

---

## 13. Suggested repository workflow

Create a separate implementation branch from the current research branch or current main as appropriate. Do not directly modify `main`.

Recommended conceptual structure if/when implementation begins:

```text
experiments/
  certaintysql/
    manifests/
    generators/
    mutations/
    baselines/
    analysis/
    fixtures/
    results/
```

Actual paths should follow existing AutoResearchClaw conventions after inspecting the repository.

The experiment implementation should use the project’s normal testing and experiment abstractions rather than introducing a parallel framework unless there is a concrete need.

---

## 14. Phase-1 only if Phase-0 passes

If misrepair is real and material, then add structured data evidence:

```text
constraint violations
conflict witnesses
query-relevant inconsistency summaries
repair-stability evidence
```

Then evaluate a diagnostic policy:

```text
QUERY_FAULT
DATA_FAULT
BOTH
INSUFFICIENT_EVIDENCE
```

Only after this should the project invest in a stronger `CertaintySQL` evidence engine using suitable CQA/range-CQA techniques.

---

## 15. Later phases

### Phase-2 — Formal certainty / range evidence

Candidate scope:

- SQLite / DuckDB first
- PK / UNIQUE
- simple FDs
- selected denial constraints
- SPJ
- GROUP BY
- COUNT / SUM / MIN / MAX
- tuple-deletion repairs
- tractable rewriting where possible
- SAT/MaxSAT fallback for selected harder cases

### Phase-3 — external validity

Move from synthetic/local tasks to controlled corruptions derived from realistic workloads such as BIRD/Spider-style schemas. Keep corruption provenance controlled; do not equate naturally dirty data with a known formal inconsistency label.

### Phase-4 — integrated database agent

Combine query diagnosis, data diagnosis, uncertainty-aware answers, repair policy, and escalation.

---

## 16. What not to do first

Avoid these until the motivating pilot passes:

- full SAT-based CQA implementation
- full Spider 2.0 integration
- broad Snowflake/BigQuery support
- complex multi-agent architecture
- learned integrity-constraint discovery
- arbitrary repair semantics
- CRUD/action-agent extensions
- large-scale model sweeps

The purpose of Phase-0 is to cheaply kill or validate the central hypothesis before expensive engineering.

---

## 17. Related research documents in this branch

Read these before implementation:

- `docs/research/CERTAINTYSQL_FEASIBILITY_NOVELTY_DEEP_DIVE_2026.md`
- `docs/research/TEXT2SQL_TOP4_DEEP_EVIDENCE_AND_PREREGISTRATION_2026.md`
- `docs/research/TEXT2SQL_NOVELTY_AUDIT_AND_PAPER_CANDIDATES_2026.md`
- `docs/research/TEXT2SQL_LITERATURE_DRIVEN_MECHANISM_TRANSFER_2026.md`
- `docs/research/TEXT2SQL_CROSS_DOMAIN_IDEA_ATLAS_2026.md`
- `docs/research/TEXT2SQL_WILD_FUSION_IDEA_ATLAS_2026.md`
- `docs/research/TEXT2SQL_AGENT_RESEARCH_MAP_2026.md`
- `docs/research/TEXT2SQL_AUTORESEARCHCLAW_INTEGRATION_PLAN.md`

The later documents should take precedence when an earlier broad idea conflicts with a later novelty audit.

---

## 18. Machine handoff prompt

A coding/research agent can start from the following instruction:

> Read the CertaintySQL research documents in `docs/research/`, especially `CERTAINTYSQL_PHASE0_EXPERIMENT_HANDOFF_2026.md` and `CERTAINTYSQL_FEASIBILITY_NOVELTY_DEEP_DIVE_2026.md`. Do not implement full CertaintySQL first. Build and validate the Phase-0 `Fault-or-Data?` pilot. Create controlled `DATA_FAULT` episodes in SQLite/DuckDB while keeping gold SQL semantically correct, run strong SQL repair baselines, measure `Misrepair Rate`, preserve complete provenance and trajectories, and apply the pre-registered go/no-go criterion. Use tests to enforce benchmark invariants and stop after the Phase-0 result and research memo unless the hypothesis is supported.

---

## 19. Research stop point

The idea-generation phase for CertaintySQL is sufficiently mature for a first empirical decision.

The next scientific action is:

```text
build the smallest controlled pilot
        ↓
measure misrepair
        ↓
allow the result to kill the hypothesis
        ↓
only then build CertaintySQL
```

This is the intended handoff point from literature/ideation work to machine-based experimentation.
