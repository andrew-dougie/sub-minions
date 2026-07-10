# The verification ladder

The full sequence for multi-cycle work, why each rung exists, and when to collapse rungs. The invariant: rungs may be collapsed or skipped, but never reordered. Verification always precedes the work that depends on it.

```
scope → map → audit → adversarial-verify → spec → execute → lead-review → independent-E2E → user-checkpoint
```

## Rungs

### 1. Scope (lead + user)
Establish what "done" means, the cycle structure, and the approval contract (when the lead stops for the user, per `checkpoints`). Ask clarifying questions only at genuine decision forks; everything answerable from the codebase gets answered from the codebase.

**Skip when:** never. Scope can be one sentence, but it exists.

### 2. Map (parallel read-only agents)
Structured maps of unfamiliar territory: components, responsibilities, data flow, invariants, every claim anchored `file:line`. Maps feed slice design for audits and Verified Context for specs. Because Verified Context is ground truth the executor must not re-check, behavioral or contract claims from a single mapper get spot-read by the lead (or passed through a skeptic) before entering a spec.

**Skip when:** the lead already knows the territory (recent work, small blast radius). Do not audit or spec into unmapped territory on the strength of assumption.

### 3. Audit (parallel read-only agents)
Sharply-scoped slices with explicit boundaries and a required finding format (severity, claim, `file:line` evidence, failure scenario). Slice edges may overlap; gaps may not.

**Skip when:** the task is construction, not investigation (a feature build against known-good territory needs mapping, not auditing).

### 4. Adversarial verify (fresh read-only skeptics)
Separate agents that did not produce the findings recommend CONFIRM or REFUTE for each load-bearing claim by quoting current code. Claims that match no quote are REFUTED (fabricated or stale); claims that quotation cannot settle — races, missing code, behavior emerging across files — come back UNPROVABLE-BY-QUOTE for the lead to judge by reading the code itself. Batch claims per skeptic (one skeptic checks several claims it didn't produce); reserve per-claim redundancy for findings that drive scheduling. Skeptics recommend and gather evidence on auditor conflicts; the lead rules. This rung exists because auditors produce confident false positives, and one refuted P1 costs less here than after it has driven a spec, an executor run, and a review.

**Skip when:** `verify=spot` (lead personally re-reads the cited lines for top findings) or `verify=off`. Default to full adversarial when findings are numerous, when severity claims drive scheduling, or when two auditors disagree.

### 5. Spec (lead)
One spec per executor per coherent change, all 8 anatomy elements (see `spec-template.md`). Every design decision is made here; none are delegated.

**Skip when:** never for delegated work. For work the lead does with its own hands, the spec collapses into the lead's own plan.

### 6. Execute (executor agents, `exec` model)
Background, batched launches, concurrency rules enforced by the lead at launch time (one writer per repo, one driver per stateful resource, frozen files named in specs).

### 7. Lead review (lead)
The lead reads every diff before it ships. Risk-proportional: state machines get scenario-by-scenario re-walks; mechanical bulk edits get pattern-plus-count verification. Deviations from spec are adjudicated: executor improvisation is rejected, spec bugs are fixed and relaunched.

**Skip when:** only if `review=off`, which is the user's call, never the lead's.

### 8. Independent E2E (test agents + optional independent review agents)
Two distinct activities, same rung:
- **Test agents** (`test` model) run scenario lists with PASS/FAIL/BLOCKED contracts. BLOCKED goes to the needs-human list after at most one retry; FAILs feed a new spec → execute → review loop.
- **Independent review** (`review=independent` only): fresh agents review the full diff before merge-readiness; their findings get adversarially verified like audit findings, then fixed through the standard loop.

**Skip when:** the change has no runtime surface worth driving (doc/comment/log-only cycles), or `review=lead` covers the review half by definition.

### 9. User checkpoint (lead + user)
Per `checkpoints`: `functional` stops before behavior changes ship; `every-cycle` stops at every cycle boundary; `none` runs to completion. A checkpoint is a full stop with a decision-ready summary: what happened, what is proposed, what is deferred, what needs human hands.

**Skip when:** `checkpoints=none`, chosen by the user.

## Collapse profiles

| Task size | Typical ladder |
|---|---|
| Tiny (one known file, known fix) | scope → spec → execute → lead-review |
| Small feature, known territory | scope → spec → execute → lead-review → E2E test |
| Medium audit-and-fix, one repo | scope → map → audit → spot-verify → spec → execute → lead-review → E2E → checkpoint |
| Large multi-repo overhaul | the full ladder, every rung, plus master doc discipline |

Two warning signs you under-collapsed (wasting lead tokens): mapping territory the session already holds; adversarially verifying a single self-evident finding. Two warning signs you over-collapsed (buying rework): an executor STOPPED on a mismatch you would have caught by mapping; a "verified" P1 dying during lead review instead of during rung 4.
