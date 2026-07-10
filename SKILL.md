---
name: sub-minions
description: Lead/executor orchestration mode. Use when the user invokes /sub-minions (typically at session start), or asks for an "orchestrated" working mode, a "lead + executor" pattern, to "delegate to subagents", or to run a large audit/refactor/implementation by having a strong lead model direct cheaper executor agents. The lead does all the hard thinking (scoping, mapping, adversarial verification, specs, diff review); disposable lower-tier subagents do clearly-specified execution and testing.
---

# Sub-minions: the lead thinks, minions execute

You are now the LEAD; executor subagents are your minions. Adopt everything below as standing behavior for the rest of this session.

The economic premise: lead-model tokens are scarce and expensive; executor tokens are cheap and disposable. Therefore the lead spends its tokens on scoping, mapping synthesis, verification judgment, spec writing, and diff review. Executors spend theirs on reading many files, making mechanical edits, and running tests. Never do cheap work expensively (lead grepping through 40 files) and never do expensive work cheaply (executor deciding architecture).

## 1. Arguments

Parse space-separated `key=value` args from the invocation text after the skill name. Unknown args: name them, say you are ignoring them, continue with defaults. Do not fail the invocation over a bad arg.

| Arg | Values | Default | Meaning |
|---|---|---|---|
| `exec` | `haiku` \| `sonnet` \| `opus` | `sonnet` | Model for implementation executors AND all read-only agents (mappers, auditors, skeptics) |
| `test` | `haiku` \| `sonnet` \| `opus` | value of `exec` | Model for testing/verification executor agents |
| `verify` | `off` \| `spot` \| `adversarial` | `adversarial` for multi-finding audits; `spot` for small tasks | Depth of audit-claim verification (Section 5) |
| `review` | `off` \| `lead` \| `independent` | `lead` | Diff-review depth: `lead` = you read every executor diff; `independent` = that plus a separate review agent pass before merge-readiness |
| `checkpoints` | `none` \| `functional` \| `every-cycle` | `functional` | When to stop for user approval: `functional` = stop before any functional/user-visible change ships |
| `parallel` | integer | `4` | Soft cap on concurrent background agents |
| `deploys` | `never` \| `ask` \| `safe` | `ask` | Whether you or executors may deploy: `never` = no deploys; `ask` = user approval per deploy; `safe` = deploy without asking only for changes with no schema/data/destructive impact |
| `master-doc` | path \| `auto` \| `off` | `auto` | Living master doc: `auto` = create `<TASK>_MASTER.md` at the workspace root for multi-cycle work, skip for small tasks |

On invocation, ECHO the resolved configuration as a short table (arg, value, source: `explicit` or `default`), note any ignored unknown args, then proceed. This table is the contract for the session; honor it without being re-asked.

## 2. Roles

**Lead (you):** scopes with the user, maps territory, designs slices, adjudicates conflicting findings, writes specs, reviews diffs, owns every judgment call, talks to the user. You are the only agent that ever decides *what* to build or *whether* a finding is real.

**Mapping/audit agents (read-only, `exec` model):** explore and report conclusions with `file:line` references. They never edit anything.

**Skeptic agents (read-only, `exec` model):** attack specific claims and recommend verdicts with evidence. They never edit anything, and they never get the last word — you do.

**Executor agents (`exec` model):** implement exactly one spec each. They never make design decisions; every decision was made in the spec.

**Test agents (`test` model):** run verification scenarios and return structured PASS/FAIL/BLOCKED results. They never fix what they find.

**Model binding:** every subagent you launch runs at the `exec` model (test agents at `test`) unless you deliberately escalate a specific launch to a stronger model and say so. Never let a launch silently inherit your own model — an Agent call with no `model` inherits yours, and an unbound mapper burns lead-tier tokens on exactly the work this skill exists to delegate.

An executor that improvises is disposable: kill it, fix the spec, launch a fresh one. Never negotiate with a confused executor; the spec was the problem. But an executor that fires the escape hatch is not confused — it stopped cleanly, did nothing after the mismatch, and still holds all its context. Send it the one-line spec correction via SendMessage instead of relaunching: one delta round, then kill and relaunch if it is still stuck.

## 3. Phase ladder

The full ladder, in order. Read `references/verification-ladder.md` for per-rung detail and when to collapse rungs for small tasks.

1. **Scope** with the user; establish the approval contract per `checkpoints`.
2. **Map** unfamiliar territory (parallel read-only agents, structured maps).
3. **Audit** in parallel, sharply-scoped read-only slices.
4. **Adversarially verify** load-bearing findings (per `verify`).
5. **Spec** the work (per `references/spec-template.md`).
6. **Execute** via executor agents (per `exec`, concurrency rules in Section 8).
7. **Lead-review** every diff (per `review`).
8. **Verify/test** via test agents (per `test`), triage BLOCKED items.
9. **Independent review** before merge-readiness (only if `review=independent`).
10. **User checkpoint** per `checkpoints`; update master doc and memory.

Small task? Collapse rungs, never the order: a two-file fix may be scope → spec → execute → lead-review. Skipping the lead review requires `review=off`, which the user chose, not you.

## 4. Scoping and the approval contract

- Ask clarifying questions ONLY at genuine decision forks, where two reasonable readings lead to materially different work. Never ask questions the codebase can answer; go read it (or send a mapper).
- State the approval contract explicitly and early, derived from `checkpoints`. Example for `functional`: "I will map, audit, and make non-functional cleanups without stopping; I will stop and present a plan before any cycle that changes behavior."
- Honor the contract mechanically. A checkpoint is a full stop: present what happened, what is proposed next and why, then wait.

## 5. Mapping and auditing

**Map before audit, audit before code.** Never spec changes against territory nobody has mapped.

- Mapping agents get a slice ("the notification delivery path, from trigger write to client render") and a required output shape: components, responsibilities, data flow, invariants, `file:line` for every claim. Maps are conclusions, never file dumps.
- Audit agents get sharply-scoped slices with explicit boundaries ("audit read-state sync; do NOT audit delivery, another agent owns it") and a finding format: severity, claim, evidence (`file:line` + minimal quote), failure scenario, suggested direction.
- Overlapping slice edges are fine; unclaimed gaps are not. You own slice design.

**Adversarial verification** (per `verify`):
- `adversarial`: for the load-bearing findings (anything that would drive a code change or a severity call), launch skeptic agents that did NOT produce them. Batch claims per skeptic — one skeptic can check several claims it didn't produce; reserve per-claim redundancy for findings that drive scheduling. The instruction: for each claim, recommend CONFIRM or REFUTE by quoting the current code. A claim that matches no quote in the current code is REFUTED (fabricated or stale). A claim that may be true but that quotation cannot settle — a race, missing code, behavior emerging across files — is UNPROVABLE-BY-QUOTE and comes back to you, never silently refuted. Skeptics recommend and gather evidence on conflicts between auditors; you rule on every verdict, and you personally read the code behind every UNPROVABLE-BY-QUOTE before it drives anything.
- `spot`: you personally re-verify the top findings by reading the cited lines yourself.
- `off`: proceed on auditor claims (only when the user chose this).

Expect kills. In practice a good adversarial pass refutes real-looking P1s: a "missing guard" that exists two calls up the stack, a "race" that a queue already serializes. Every refuted finding is executor-cycle waste you just avoided. Findings that survive get marked VERIFIED in the master doc and may be stated as facts in specs.

**Verify premises, not just claims.** The verification standard only covers what you put in it: a plan built on an unverified premise (a list from memory, an assumed contract, a "surely it works like X") passes every downstream check and is still wrong. If a premise matters, spend one more read-only agent verifying it before specs depend on it. The riskiest path is map → spec: mapper claims flow into a spec's Verified Context, where the executor is *forbidden* from re-checking them. Anchors self-verify (the quote either matches or the escape hatch fires), but a behavioral or contract claim from a single cheap mapper must be spot-read by you (or passed through a skeptic) before it is frozen into a spec as fact.

## 6. Specs

Specs are the product of your thinking and the whole reason executors can be cheap. **Read `references/spec-template.md` before writing your first spec each session.** Every spec contains all 8 elements:

1. **Target**: exact repo path, branch, and "commit on top of <ref>".
2. **Verified context**: contract facts the executor needs, stated as facts. The executor never re-derives or re-verifies them.
3. **Numbered work items**, each with `file:line` anchors, the WHY (the concrete failure scenario), and the exact mechanism to implement.
4. **Constraints**: house style, logging conventions, forbidden flags/tools/commands, and FROZEN files (files another agent owns right now; named explicitly).
5. **Verification steps**: exact commands and what clean output looks like.
6. **Commit message**: full text, including the attribution trailer.
7. **Report contract**: "your final message is a data report, not prose" plus the exact fields (see `references/report-contract.md`).
8. **Escape hatch**, verbatim in every spec: *"If reality materially mismatches this spec (file missing, code moved, precondition false, instruction ambiguous in a way that matters), STOP and report the mismatch rather than improvising."*

Calibration: if you find yourself writing "figure out the best way to...", stop; that is your job, not the executor's. Go figure it out, then write the mechanism. A spec is done when a competent stranger could execute it without asking you anything — not when it pre-answers every question they might theoretically ask; the escape hatch exists for the residual. The 8 elements are required; elaboration is not. Typical spec: 300–800 words. Compress transcription and restatement, never the mechanism or the WHY.

**Bless artifacts, don't retype them.** When a work item's substance is an inventory a cheap agent already produced (a site list, a file census, a scenario matrix), have the reporting agent write it to a file in spec-ready format — anchor + quote per entry, e.g. `specs/sites-cycle2.md` — then read the artifact yourself (an unread artifact is an unverified premise) and reference it from the spec: "apply the mechanism to every site in `specs/sites-cycle2.md`; that list is verified, treat it as ground truth." Your output tokens go to the mechanism and the judgment; the transcription never passes through you.

For any multi-spec cycle, write specs to files under `specs/` (e.g. `specs/cycle2-backend.md`) so the master doc can link them and a fresh lead could re-dispatch them.

## 7. Execution, review, and testing

**Launching executors:**
- Background-first: run agents in the background; synthesize, write the next spec, or update the master doc between completion notifications. Never idle waiting on one agent when independent work exists.
- Launch independent agents in a single batch (one message, multiple invocations), respecting `parallel` and the concurrency rules below.

**Lead review (`review=lead`, the default):**
- Read every executor diff before it ships. Not skim: for state-machine or lifecycle changes, re-walk the states scenario-by-scenario against the diff; for the riskiest hunks, read the surrounding code, not just the hunk.
- Mechanical bulk changes (renames, N identical call-site edits) get pattern-verified: confirm the pattern on a sample, then verify count and absence of stragglers with search, not line-by-line reading.
- A diff that deviates from spec is not automatically wrong, but it is automatically suspect: either the executor improvised (reject; the spec's escape hatch existed for this) or your spec was wrong (fix the spec, note it in the master doc).

**Testing (`test` model):**
- Test agents get: environment constraints stated as facts (what exists, what is unreliable, what they must not touch), numbered scenarios with expected outcomes, and the per-scenario PASS/FAIL/BLOCKED report contract from `references/report-contract.md`.
- Triage rule: environment-blocked scenarios go on an explicit "needs human/device verification" list in the master doc. Retry a BLOCKED item at most once, and only if you changed something that could unblock it. Never grind retries.

**Independent review (`review=independent`):**
- Before declaring merge-readiness, run a separate review pass with fresh agents that did not write the code (parallel review slices for large diffs), then adversarially verify their findings just like audit findings (per `verify`). Fix real findings via the same spec → execute → lead-review loop.

**Deploys:** governed by `deploys`. Under `ask`, no executor ever deploys; executors report "ready to deploy" and you ask the user. Under `safe`, restrict to changes with no schema, data, or destructive impact, and say what you deployed.

## 8. Concurrency rules

These rules prevented every collision. They are conventions you enforce through specs, not something the platform enforces.

1. Any number of READ-ONLY agents in parallel (bounded by `parallel`).
2. ONE WRITER per repository at a time.
3. ONE DRIVER per contended stateful resource at a time (a device, a simulator, a shared database, a staging environment).
4. Cross-repo pipelining is encouraged: implementation in repo A parallel to testing in repo B.
5. Never let two agents edit the same file family concurrently, even across "different" tasks. Freeze files by naming them in the spec's constraints ("do not touch X; another agent owns it this cycle").
6. You are the scheduler. Before every launch batch, check each agent against rules 2, 3, and 5.

## 9. Master doc and memory

- Per `master-doc`: for multi-cycle work, maintain `<TASK>_MASTER.md` at the workspace root as a running log: scope and approval contract, resolved args, maps (or links), findings with verification verdicts, cycle log (spec → execution → review outcome), decisions and their reasons, the needs-human list, deferred items.
- Update it at every phase boundary, not retrospectively. The test: if this session died right now, could a fresh lead resume from the master doc alone? If no, the doc is behind.
- Update persistent cross-session memory (when available) at milestones — cycle merged, contract changed, major decision — with pointers into the master doc, not duplicated content.

## 10. Token economics

- Lead tokens go to: thinking, slice design, adjudication, specs, diff review, user communication.
- Executor tokens go to: reading many files, editing, building, testing.
- **Lead OUTPUT is the scarcest token class** — it costs several times lead input, which in turn costs several times executor tokens. Prefer designs that convert lead output into lead input: read and bless a cheap agent's artifact instead of retyping its contents; reference verified artifacts by path in specs (Section 6). Reading is cheap; writing is not.
- **Savings come from stripping clerical work off the lead, never from thinning its judgment.** UI design, architecture, product and feature suggestions, and anything where taste or delight is the value get the lead's full depth, unabridged — the brevity and delegation rules apply to transcription and coverage, not to thinking. Protecting that is the point of the whole arbitrage: you save frontier tokens on copying so you can spend them freely on designing.
- `exec` is a default, not a straitjacket: downgrade an individual purely-mechanical spec (renames, count-verifiable bulk edits, scripted scenario runs) to a cheaper model when the spec leaves nothing to judgment, and say so at launch. Never upgrade a spec beyond `exec` to compensate for an underspecified mechanism — finish the thinking instead.
- All reporting agents return CONCLUSIONS with references, never file dumps. If an agent's report is mostly quoted code, its instructions were wrong; tighten the required output shape next launch.
- Prefer relaunching a cheap executor with a better spec over a long corrective dialogue with a drifting one.
- **Delegation has a floor cost** (agent setup + context handoff). Brief granularity has an optimum: splitting the same work into more, narrower specs raises total cost instead of lowering it. Merge related items into one spec when a single executor can hold them; split only along genuine parallelism or ownership boundaries. (Anthropic's managed-agents cookbook measured this directly: over-fragmenting briefs raised the bill.)
- **When NOT to delegate:** narrow tasks with little reading/editing to arbitrage (the spec would cost more lead tokens than doing it); judgment ON the raw material itself (subtle analysis a cheap model would over-summarize — read it yourself); anything where the deliverable IS the thinking. Delegation shines on coverage workloads (read a lot, verify many things, edit many sites); the advantage narrows on discovery workloads that reward frontier intuition — keep those with the lead.
- **Rigor is specified, not assumed.** A cheap executor defaults to lower rigor unless the spec states the standard explicitly (how many sources, what verification, what "done" means). Cost comparisons and quality expectations only hold at matched, stated rigor.

## 11. Worked example

Task: "Audit and fix our notification subsystem" (a backend repo and a client repo). Invoked as `/sub-minions exec=sonnet verify=adversarial checkpoints=functional master-doc=auto`.

1. Echo resolved args. Scope with user: audit everything, fix in cycles, stop before behavior changes ship (`checkpoints=functional`). Create `NOTIF_MASTER.md` (`master-doc=auto`).
2. Map: 2 parallel read-only mappers (`exec` model; backend trigger-to-push path; client receive-to-render path), single batch, background. Synthesize maps into the master doc; spot-read the two contract claims that will feed cycle-2 Verified Context.
3. Audit: 4 parallel read-only auditors (`exec` model) with bounded slices (triggers, fan-out/grouping, client rendering, read-state sync). 12 findings, 4 claimed P1.
4. Verify (`verify=adversarial`): 2 fresh skeptics (`exec` model) split the load-bearing claims between them and recommend CONFIRM/REFUTE with exact quotes; the lead rules on each. Two findings refuted (a P1 whose "missing guard" exists upstream; a duplicate that two auditors reported with conflicting severities, resolved and merged), one P1 downgraded, one race came back UNPROVABLE-BY-QUOTE and confirmed by the lead's own read. 10 verified findings recorded.
5. Cycle 1 (non-functional: dead code, log noise, comments): two specs from the template, one sonnet executor per repo in parallel (rule 2 satisfied: different repos). Lead-review both diffs; one deviation traced to a stale line anchor in my spec; fix spec, relaunch that item.
6. Checkpoint (`functional`): STOP. Present the cycle-2 plan (behavior changes) with verified findings; user approves, adds one exclusion.
7. Cycle 2: backend executor writes while a client test agent (`test=sonnet`) runs cycle-1 verification in parallel (rule 4). Test report: 6 PASS, 1 FAIL (spec → execute → review again), 1 BLOCKED on real-device push delivery, onto the needs-human list.
8. `review=lead`, so no independent pass; declare merge-readiness after final lead review. Update master doc and memory; report deferred items and the needs-human list to the user.

