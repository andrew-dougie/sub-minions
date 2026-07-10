# Report contracts

Two required formats: the executor (implementation) report and the verification (testing) report. Paste the relevant block into every spec's Report section. The point of both: the lead machine-reads dozens of these; prose narration wastes lead tokens and hides omissions. A section with nothing to report is written as `none`, never omitted, so silence is always meaningful.

## Executor report (implementation agents)

Final message shape, exactly:

```
STATUS: COMPLETE | PARTIAL | STOPPED

COMMITS:
- <sha> <subject line>

FILES:
- <path of every file touched, one per line>

ITEMS:
- Item 1: DONE | SKIPPED | STOPPED — <one line: what was done, or why not>
- Item 2: ...

VERIFICATION:
- <command>: PASS | FAIL — <one-line result, e.g. "exit 0, 0 warnings" or the first error line>

DEVIATIONS:
- <anything not exactly per spec, one line each, with reason> | none

OBSERVATIONS:
- <at most 3 bullets: things noticed but deliberately NOT acted on> | none
```

Semantics:

- `COMPLETE`: every item done, every verification command passing.
- `PARTIAL`: some items done and committed; remaining items listed with STOPPED/SKIPPED and reasons. Never commit half an item.
- `STOPPED`: escape hatch fired. Report the exact mismatch: what the spec said, what reality is (with `file:line` and a short quote), and nothing else attempted after the mismatch.
- DEVIATIONS is the honesty channel. A deviation reported is a spec bug the lead can fix; a deviation hidden becomes a review-phase incident. Executors are never penalized for reporting deviations or stopping.
- FILES is the lead's review index: the diffs to read and the list to check against this cycle's frozen files, with no discovery pass.
- OBSERVATIONS lets an executor surface adjacent problems without scope-creeping into them. An observation the lead later acts on re-enters the pipeline as an UNVERIFIED claim — it gets verified per `verify` before driving a spec, never promoted straight to fact.

## Verification report (test agents)

One verdict per scenario, no aggregate hand-waving. Final message shape, exactly:

```
ENVIRONMENT: <what was actually used: device/sim/emulator/stage, versions, accounts>

SCENARIOS:
- S1 <name>: PASS — <one line of evidence: what was observed>
- S2 <name>: FAIL — <expected X, observed Y; exact error/log line if available; repro count, e.g. "2/2 attempts">
- S3 <name>: BLOCKED — <the environmental reason; what would unblock it>

ARTIFACTS: <paths to screenshots/logs/recordings> | none

NOT TESTED: <scenarios from the spec not attempted, with reason> | none
```

Semantics:

- `PASS` requires observed evidence, not absence of error. "It didn't crash" is not a PASS for "the badge updates".
- `FAIL` must contain expected vs. observed and be reproducible enough to hand to a fix cycle (repro steps or count).
- `BLOCKED` means the environment cannot exercise the scenario (missing hardware, unreachable service, permission wall). It is a routing verdict, not a failure: the lead moves it to the needs-human list. Test agents must not retry a BLOCKED scenario more than once, and must not downgrade a FAIL to BLOCKED to look cleaner.
- NOT TESTED covers scenarios skipped for non-environmental reasons (time cap hit, earlier blocker made them moot). Every scenario in the spec appears in exactly one of SCENARIOS or NOT TESTED.
