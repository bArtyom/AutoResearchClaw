# DeltaSQL / Text2CRUD Semantic-Effect Verification — Deep Novelty Audit (2026-08-24)

> Status: fresh literature-first spike; no implementation changes.
> Goal: determine whether “natural-language CRUD intent → state-transition contract → verified database mutation” survives the closest 2024–2026 database-agent, tool-verification, formal-specification, and transaction-safety work.

## 1. Executive conclusion

The broad idea **survives only in a narrower form**.

The following claims are already occupied and should not be used as the paper thesis:

- agents should execute risky writes on branches/sandboxes before committing;
- agent tool calls should have pre/postconditions;
- natural language can be translated into formal postconditions;
- state-changing agent actions should be checked against a task specification;
- database-agent success should be evaluated from the post-action system state;
- LLM-generated transactions may contain semantic errors and require special rollback/removal handling.

The defensible database-specific gap is:

> **Can a system compile a natural-language database mutation request into a relational state-delta specification that is both discriminative enough to catch semantically wrong SQL workflows and permissive enough to accept multiple valid implementations, then verify that specification against the actual pre/post database states?**

The key scientific object is the **relational effect specification**, not branching or generic runtime verification.

A stronger name is:

> **RelSpec: Intent-to-Relational-Effect Verification for Text2CRUD Agents**

`DeltaSQL` can remain an implementation/system name.

---

# 2. Closest collision 1 — BIRD-INTERACT already evaluates full CRUD with executable tests

BIRD-INTERACT (ICLR 2026 Oral) explicitly moves beyond SELECT-only Text2SQL. It includes BI and full CRUD tasks, dynamic user interaction, database exploration, preprocessing SQL, cleanup SQL, and executable test cases.

References:
- Paper: https://proceedings.iclr.cc/paper_files/paper/2026/hash/496b549556509bbb9770bf9d335c5800-Abstract-Conference.html
- Repository: https://github.com/bird-bench/BIRD-Interact

Its task representation already includes fields such as:

```text
query
amb_user_query
sol_sql
preprocess_sql
clean_up_sql
test_cases
follow_up
external_knowledge
```

Therefore:

> “evaluate CRUD agents by running tests after database mutations”

is not new.

## Surviving difference

BIRD-INTERACT uses benchmark-authored executable test cases to judge whether the task was solved. It does not make **automatic intent-to-effect specification** the central research problem.

RelSpec would ask whether the natural-language request itself can be compiled into a reusable, database-level contract that exposes:

- required effects;
- forbidden effects;
- cardinality bounds;
- preservation obligations;
- idempotency expectations;
- cross-table invariants;
- acceptable alternative implementations.

The contract should be executable without knowing the gold SQL.

---

# 3. Closest collision 2 — DBA-Bench already has outcome-first, safety-constrained state evaluation

DBA-Bench (July 2026) is an especially important collision. It evaluates database-operation agents on live PostgreSQL, persistent state, active workloads, and open-ended remediations. It explicitly defines success through **scenario-specific post-run outcome verifiers** and separately checks operational safety.

Reference: https://arxiv.org/abs/2607.22165

Important consequence:

> “judge database agents by the state they produce rather than their action text”

is also occupied.

DBA-Bench even motivates multiple valid remediation paths: different actions may be acceptable if they satisfy the same scenario-specific success contract.

## Surviving difference

DBA-Bench’s success contracts are scenario-authored benchmark infrastructure for DBA operations such as recovery, tuning, schema change, and fault remediation.

RelSpec would study a different translation problem:

```text
natural-language business mutation request
                 ↓
automatically synthesized relational effect specification
                 ↓
verify arbitrary SQL workflow against actual pre/post state
```

The system must infer the contract rather than receive a manually authored scenario verifier.

This moves the paper closer to **specification synthesis for relational state** than to another database-agent benchmark.

---

# 4. Closest collision 3 — ToolGate already uses Hoare-style pre/postconditions

ToolGate (ACL Findings 2026) gives every tool a Hoare-style contract. A symbolic state records trusted world information; preconditions gate invocation, and postconditions decide whether the returned result may update the trusted state.

Reference: https://aclanthology.org/2026.findings-acl.470/

Therefore:

> “add preconditions and postconditions to database tools”

is not new.

## Surviving difference

ToolGate assumes tool contracts describe the **tool**.

RelSpec needs a task-specific contract describing the **user-requested database transformation**.

A generic `UPDATE` tool might have a valid ToolGate contract and still perform the wrong business mutation.

Example:

```text
Tool contract:
  UPDATE is permitted and returned success.

User request:
  activate customer 42, preserving billing history,
  and create exactly one audit row.

Executed workflow:
  activates customer 42
  deletes previous billing state
  creates two audit rows
```

The tool execution is valid; the task effect is not.

---

# 5. Closest collision 4 — Natural-language intent → formal postcondition is a mature emerging field

The software-engineering literature already directly studies translation from natural language into programmatically checkable postconditions.

Key references:

- Endres et al., *Can Large Language Models Transform Natural Language Intent into Formal Method Postconditions?*, FSE 2024. https://nl2postcond.github.io/
- Faria et al., *Automatic Generation of Formal Specification and Verification Annotations Using LLMs and Test Oracles*, 2026. https://arxiv.org/abs/2601.12845
- Leite et al., *Generating formal smart-contract specifications*, Science of Computer Programming 2026.
- AutoSpec+, ACL Demo 2026. https://aclanthology.org/2026.acl-demo.66/
- POSTCONDBENCH, ACL Findings 2026. https://aclanthology.org/2026.findings-acl.1768/
- SpecMind, ACL 2026. https://aclanthology.org/2026.acl-long.1687/

Thus:

> “use an LLM to convert natural language into formal postconditions”

is not novel either.

## What makes relational effects structurally different

Database mutation specifications are not ordinary method postconditions. They often quantify over **sets/bags of rows and relations** and must distinguish intended local change from unintended global change.

Examples:

```text
exactly one account with id=42 changes status

no existing invoice tuple changes

for every order touched, exactly one audit tuple is added

sum(balance) is conserved across an account transfer

all pre-existing rows outside tenant T remain identical

repeating the request produces no additional state change
```

The output state is also observable and queryable, enabling exact differential checks over relational snapshots.

The research question becomes whether a relational DSL and verifier can exploit this structure to outperform generic postcondition generation.

---

# 6. Closest collision 5 — RefineAct already derives task specifications and runtime-verifies agent actions

RefineAct (ASE 2026) is a strong generic-agent collision. It derives a task-specific formal specification from the user instruction, translates it into Prolog predicates, refines the specification into action-level pre/postcondition chains, and verifies proposed actions at runtime.

Reference: https://lab-design.github.io/papers/ASE-26/

Therefore the claim

> “derive a task-specific spec from natural language and block unsafe actions”

is occupied at the general-agent level.

## Surviving difference

RelSpec must demonstrate a **database-specific advantage**:

1. relational before/after states are exact and large;
2. mutations may have hidden effects through triggers/cascades;
3. multiple SQL programs can induce the same valid state transition;
4. SQL text is a weak evaluation target, while relational deltas are implementation-invariant;
5. database constraints and query execution can automatically verify many obligations;
6. bag/set semantics and aggregate conservation rules create verification opportunities absent from generic symbolic action state.

Without a measurable gain from exploiting relational structure, RelSpec would be merely a domain adaptation of RefineAct.

---

# 7. Closest collision 6 — Intent2Tx and DeFi intent-alignment show an adjacent state-transition benchmark

Intent2Tx (2026) translates natural-language intents into Ethereum transactions and evaluates them using **differential state analysis on forked mainnet environments**. Its authors explicitly note that syntactically valid actions often fail to achieve the intended state transition.

Reference: https://arxiv.org/abs/2604.27763

Other 2026 DeFi work also verifies intent-to-transaction alignment before funds move.

This is conceptually close to RelSpec.

## Surviving difference

Relational databases provide a different and arguably more general effect algebra:

- arbitrary table sets rather than protocol-specific contract state;
- SELECT/INSERT/UPDATE/DELETE/DDL workflows;
- SQL triggers, cascades, views, constraints, and transactions;
- row-level over/under-mutation;
- aggregate invariants;
- many equivalent SQL implementations;
- cross-tenant and governance boundaries.

The paper should cite Intent2Tx as a close conceptual precedent, not pretend intent-to-state verification is new.

---

# 8. Closest collision 7 — LLM-generated transaction error handling already exists

Zeng, Wu, and Krishnan (2024) study semantic errors in LLM-generated database transactions and use Invariant Satisfaction/I-Confluence to coordinate suspicious/removable transactions while preserving database consistency.

Reference: https://arxiv.org/abs/2412.12493

This occupies:

- LLM transactions can be semantically wrong;
- database invariants matter;
- special infrastructure may be needed to undo or buffer them.

## Surviving difference

Their focus is preserving **database consistency while transactions may later be removed/reviewed**.

RelSpec focuses on a prior question:

> Did the mutation actually implement the user’s requested relational effect?

A transaction can satisfy all schema/database invariants and still be semantically wrong relative to user intent.

---

# 9. Exact surviving research problem

After all collisions, the strongest formulation is:

## RelSpec: Natural-Language-to-Relational-Effect Specification

Input:

```text
- natural-language mutation intent I
- schema S
- optional business rules K
- pre-state database D0
```

Output:

```text
RelSpec R = {
  required_effects,
  forbidden_effects,
  preservation_constraints,
  cardinality_constraints,
  aggregate_constraints,
  idempotency_constraints,
  relational invariants,
  uncertainty / unresolved assumptions
}
```

Given a candidate SQL workflow `P`:

```text
D1 = execute(P, D0)
```

verify:

```text
R(D0, D1) = true / false / unknown
```

The important property is **implementation invariance**:

Two very different SQL workflows should both pass if they produce an acceptable requested state transition.

---

# 10. Candidate relational-effect DSL

A useful V0 DSL should stay deliberately small.

```yaml
scope:
  tenant_id: 17

require:
  update:
    table: customers
    where: customer_id = 42
    changes:
      status:
        from: trial
        to: active
    count: 1

  insert:
    table: activation_audit
    where:
      customer_id: 42
    count: 1

preserve:
  - table: invoices
    where: customer_id = 42
    projection: '*'

forbid:
  - table: customers
    where: customer_id != 42
    any_change: true

invariants:
  - expr: SUM(account.balance)@before == SUM(account.balance)@after

idempotency:
  repeat_effect: no_additional_change
```

Possible deterministic verifier primitives:

- relation diff;
- tuple insert/delete/update sets;
- projection equality;
- cardinality/count checks;
- aggregate before/after comparison;
- key/FK/check-constraint evaluation;
- trigger/cascade-observed effect accounting;
- repeat-execution delta.

---

# 11. The paper should separate two subproblems

## Task A — Specification synthesis

Can the system infer the correct RelSpec from natural language?

Metrics:

- obligation precision/recall;
- discriminative power against wrong mutation programs;
- overconstraint rate (rejects valid alternative workflows);
- underconstraint rate (accepts harmful workflows).

## Task B — Mutation verification

Given a correct RelSpec, can the deterministic relational verifier reliably identify:

- exact intended changes;
- over-mutation;
- under-mutation;
- forbidden side effects;
- invariant violations;
- non-idempotent retries?

Task B should be nearly deterministic. Most modeling uncertainty belongs in Task A.

This separation prevents an LLM verifier from hiding specification errors.

---

# 12. Benchmark design: mutation pairs, not only gold SQL

A strong benchmark should contain **equivalence classes of valid and invalid mutation programs**.

For one intent:

```text
P_valid_1: direct UPDATE + INSERT
P_valid_2: CTE-based workflow
P_valid_3: stored procedure call

P_invalid_1: correct target + extra row modified
P_invalid_2: correct visible value + audit omitted
P_invalid_3: correct target + historical rows deleted
P_invalid_4: duplicate audit insert
P_invalid_5: correct first execution, wrong retry behavior
P_invalid_6: valid SQL but cross-tenant side effect
```

This enables evaluation of **discriminative completeness** without rewarding one reference SQL syntax.

### Mutation operators for hard negatives

- WHERE predicate broadening/narrowing;
- missing tenant filter;
- wrong join during UPDATE ... FROM;
- duplicate insert;
- omission of secondary required effect;
- unintended cascade;
- overwrite history instead of append;
- wrong NULL handling;
- non-idempotent retry;
- partial multi-table commit;
- stale-read conditional update;
- trigger-induced extra mutation.

---

# 13. Most important baseline set

RelSpec cannot be compared only against a raw LLM.

Required baselines:

1. BIRD-style benchmark-authored executable tests;
2. post-state LLM judge with NL request + SQL + diff;
3. SQL AST safety rules;
4. row-count / touched-table guards;
5. generic NL→postcondition generation;
6. ToolGate-style fixed tool contracts where applicable;
7. RefineAct-style task specification/runtime verification if reproducible;
8. oracle relational contract.

The oracle contract ceiling is critical: if even perfect relational contracts add little over existing executable tests, the research problem is weak.

---

# 14. Killer experiment 0 — does hidden over-mutation exist beyond existing tests?

Before implementing a full specification synthesizer:

1. Sample CRUD tasks from BIRD-Interact or construct equivalent local tasks.
2. Generate semantically plausible mutation programs that pass basic syntax/permission checks.
3. Measure whether benchmark tests catch all harmful over/under-mutations.
4. Add hand-authored oracle RelSpecs.
5. Measure incremental detection.

Go/no-go criterion:

> If oracle RelSpecs detect less than ~10% additional semantically harmful mutations beyond strong existing task tests, the benchmark contribution is probably too small.

If the oracle gap is large, proceed to automatic RelSpec synthesis.

---

# 15. Killer experiment 1 — specification discriminative power

Given each intent and a set of valid/invalid mutation implementations:

```text
NL intent → generated RelSpec → classify mutation as acceptable/unacceptable
```

Primary metric:

```text
balanced accuracy over valid alternative implementations and semantic mutants
```

But separately report:

- false rejection of valid alternatives;
- false acceptance of harmful mutations.

This is more meaningful than string matching the generated specification to a reference.

---

# 16. Falsification conditions

Stop or substantially downgrade the direction if any of the following holds:

1. strong existing executable tests already detect nearly all harmful state changes;
2. generic postcondition-generation methods perform as well as the relational DSL;
3. generated RelSpecs overconstrain valid SQL alternatives at an unacceptable rate;
4. benchmark mutants are trivially detected by touched-table/row-count heuristics;
5. the LLM cannot resolve enough intent to produce usable contracts without simply reproducing a hidden gold SQL;
6. real CRUD tasks rarely contain meaningful preservation/forbidden-side-effect obligations beyond existing DB constraints.

---

# 17. New derived ideas from this audit

## R1. Relational PostconditionBench

A database-specific analogue of POSTCONDBENCH: natural-language mutation intents paired with expert relational postconditions and multiple valid/invalid SQL workflows.

## R2. Effect Mutation Testing for Data Agents

Evaluate a verifier by mutation score over semantic SQL-state mutants, not just benchmark task accuracy.

## R3. Over-Mutation Rate

New agent metric:

```text
fraction of changed relational state not justified by the requested effect
```

## R4. Under-Mutation Rate

Fraction of required state-transition obligations left unsatisfied.

## R5. Semantic Idempotency Benchmark

Replay natural-language CRUD requests under retries/timeouts and test whether agents accidentally duplicate effects.

## R6. Intent-Preserving Transaction Equivalence

Determine whether two mutation workflows are equivalent *with respect to a user intent*, which may be weaker than full database-state equivalence.

## R7. Trigger-Aware Effect Verification

The SQL statement text may not reveal all effects. Verify actual trigger/cascade effects against the intent contract.

## R8. Concurrent RelSpec

Extend contracts with allowed behavior under concurrent updates, e.g. optimistic concurrency/version constraints.

## R9. Minimal Sufficient Contract

Search for the smallest set of relational obligations that rejects all known invalid mutants while accepting valid alternatives. This mirrors specification inference and avoids verbose brittle contracts.

## R10. Contract-Guided SQL Repair

If verification fails, return the violated state-effect obligation rather than a generic SQL error. Repair only the responsible effect.

---

# 18. Revised judgment

The earlier `DeltaSQL` idea was too broad because 2026 already has:

- CRUD Text2SQL evaluation;
- outcome-first database-agent evaluation;
- branchable agent environments;
- Hoare-style tool contracts;
- natural-language postcondition synthesis;
- generic runtime intent formalization;
- state-differential evaluation in DeFi;
- LLM transaction semantic-error handling.

The narrower **RelSpec** problem remains interesting because it combines these ingredients around an under-specified object: **relational state effects induced by a natural-language data mutation request**.

Revised score:

- Novelty: 4/5, not 5/5
- Feasibility: 4/5
- Scientific clarity: 5/5
- Collision risk: medium
- Best initial contribution: benchmark + relational-effect DSL + oracle-gap study
- Only after the oracle-gap study: automatic NL→RelSpec synthesis and branch-integrated verifier

Current recommendation: **keep as a top candidate, but require an oracle-gap pilot before any large implementation.**
