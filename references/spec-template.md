# Executor spec template

Copy everything below the divider into a new spec (inline in the agent prompt, or a file under `specs/` for multi-spec cycles or non-subagent platforms). Replace ALL-CAPS placeholders. The `<!-- ... -->` comments are guidance for the lead writing the spec; delete them before sending, or leave them, executors ignore them either way.

Rules of thumb while filling this in:

- Every fact you state in Verified Context must actually be verified (by a mapper, a skeptic, or your own reading of the cited lines). The executor will build on it without checking.
- If a work item says "figure out" or "choose the best", the spec is not done. Decide, then write the mechanism.
- Line numbers drift. Anchor with `file:line` PLUS a short unique code quote so the executor can relocate the site if lines moved. A moved-but-findable anchor is fine; a missing one triggers the escape hatch.
- The spec is done when a competent stranger could execute it without asking you anything.

---

# SPEC: SHORT-TASK-TITLE

<!-- One spec = one executor = one coherent commit (or small commit series). Split unrelated work into separate specs. -->

## 1. Target

- Repo: /ABSOLUTE/PATH/TO/REPO
- Branch: BRANCH-NAME  <!-- executor checks out this exact branch; say whether to create it and from where -->
- Base: commit on top of REF-OR-SHA. Do not rebase, do not merge, do not touch other branches.

## 2. Verified context (facts, do not re-derive)

<!-- Contract facts the executor needs. Each one was verified upstream (map, adversarial pass, or your own read). State them flatly; the executor treats them as ground truth. 3-10 bullets typical. -->

- FACT ABOUT DATA FLOW OR CONTRACT, e.g. "FooService.push() is the only write path to the `events` table; callers at `src/a.ts:88` and `src/b.ts:141`."
- FACT ABOUT AN INVARIANT, e.g. "`state` is only ever `pending|sent|failed`; `sent` is terminal."
- FACT ABOUT THE ENVIRONMENT, e.g. "The integration tests require the local emulator; unit tests do not."

## 3. Work items

<!-- Numbered. Each item: WHERE (file:line + short quote), WHY (the concrete failure scenario, so the executor understands intent when code has drifted), WHAT (the exact mechanism: names, signatures, control flow). -->

### Item 1: SHORT NAME

- Where: `path/to/file.ext:123`, at `unique short code quote`
- Why: FAILURE SCENARIO, e.g. "When X arrives before Y, the handler reads Z as nil and drops the event silently."
- What: EXACT MECHANISM, e.g. "Guard at the top of `handle()`: if `z == nil`, enqueue via existing `retryQueue.add(event)` and return. Do not add a new queue. Log via HOUSE-LOGGER at warn level with the event id."

### Item 2: SHORT NAME

- Where: ...
- Why: ...
- What: ...

## 4. Constraints

<!-- House rules and freezes. The freeze list is load-bearing for concurrency: name every file another agent owns this cycle. -->

- Style: FOLLOW EXISTING FILE STYLE; note anything project-specific (naming, error handling idiom).
- Logging: use HOUSE-LOGGING-MECHANISM only; never PRINT/CONSOLE.LOG equivalents.
- Forbidden: FLAGS/TOOLS/COMMANDS THE EXECUTOR MUST NOT USE, e.g. "no --force flags, no dependency additions, no formatter runs over unrelated files."
- FROZEN files (another agent owns these right now; do not read-modify-write them): `path/one.ext`, `path/two.ext`
- Scope: touch only files named in work items plus their test files. Anything else: escape hatch.

## 5. Verification (run before committing)

<!-- Exact commands and what clean looks like. If output is nontrivial to judge, describe the pass signal precisely. -->

1. `BUILD-OR-TYPECHECK-COMMAND` — expect: EXIT 0 / "Build succeeded" / zero new warnings.
2. `TEST-COMMAND` — expect: ALL PASS; the N pre-existing failures in SUITE-NAME are known, ignore them.
3. OPTIONAL TARGETED CHECK, e.g. `grep -rn 'OLD_SYMBOL' src/` — expect: no hits.

## 6. Commit

Single commit (or one per work item if items are independent) with exactly this message:

```
COMMIT-SUBJECT-LINE

OPTIONAL-BODY-LINES

REQUIRED-ATTRIBUTION-TRAILER-LINE
```

<!-- Replace REQUIRED-ATTRIBUTION-TRAILER-LINE with your platform/org's trailer verbatim, e.g. "Co-Authored-By: Agent Name <noreply@host.com>". Do not let the executor compose its own commit message. -->

## 7. Report

Your final message is a data report, not prose. Use exactly this shape (full format in references/report-contract.md):

- STATUS: COMPLETE | PARTIAL | STOPPED
- COMMITS: sha + subject, one line each
- PER-ITEM: item number → done/skipped + one line (what was done, or why skipped)
- VERIFICATION: each command → result verbatim summary
- DEVIATIONS: anything not exactly per spec, with one-line reason (empty section if none)
- OBSERVATIONS: at most 3 bullets of things noticed but NOT acted on

## 8. Escape hatch

If reality materially mismatches this spec (file missing, code moved, precondition false, instruction ambiguous in a way that matters), STOP and report the mismatch rather than improvising.
