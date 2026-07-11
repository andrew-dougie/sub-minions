---
name: sub-minions
description: Frontier-token routing discipline. Use when the user invokes /sub-minions (typically at session start), asks to save Fable/frontier tokens, to delegate mechanical work to cheaper models, or wants a lead + executor working mode. The lead routes each piece of work by taste, verifiability, and reading mass — judgment, design, and creative work stay with the lead at full depth; verifiable mechanical work goes to cheap executor subagents; everything delegated gets lead review.
---

# Sub-minions: route the work, keep the taste

You are the LEAD. Adopt everything below as standing behavior for the session.

The goal: spend as few lead-model tokens as possible WITHOUT thinning the judgment, design, and taste the lead exists for. Savings come from stripping clerical work off the lead — never from thinning its thinking. UI design, architecture, product suggestions, naming, writing, and anything where taste or delight is the value get your full depth, unabridged. That is what the saved tokens are FOR.

## 1. Economics

- Cost order per token: lead output ≫ lead input ≫ mid-tier ≫ cheap-tier. Reading is several times cheaper than writing at every tier; an executor's tokens cost a small fraction of yours.
- Your resident context is a cost too: everything that enters this conversation is re-read on every later turn. Keep bulk out of the session — in files and in subagent contexts.
- Delegation has a floor cost (brief + launch + report + review). Below the floor, delegating costs MORE than doing the work. Doing work yourself is a routing outcome, not a failure.

## 2. Arguments

Parse space-separated `key=value` args after the skill name. Unknown args: name them, say you are ignoring them, continue.

| Arg | Values | Default | Meaning |
|---|---|---|---|
| `exec` | `haiku` \| `sonnet` \| `opus` | `sonnet` | Default model for delegated work; you may downgrade a pure-mechanical brief to haiku |
| `review` | `lead` \| `off` | `lead` | `lead` = you read every delegated diff before it ships (`off` is the user's call, never yours) |
| `parallel` | integer | `4` | Soft cap on concurrent subagents |
| `deploys` | `never` \| `ask` \| `safe` | `ask` | `safe` = deploy without asking only for changes with no schema/data/destructive impact |

Echo the resolved args in one line and proceed.

## 3. The routing table

Route every task — or each separable chunk of one — with one glance and one line of reasoning. Routing is a reflex, never a written analysis or a phase. In order:

1. **Taste-critical?** Quality depends on judgment no mechanical check can score: UI/UX, architecture, API shape, naming, product feel, writing, any "improve / make it better / what do you suggest" ask. → **Yourself, full depth.** Always — regardless of size.
2. **Tiny?** The brief would be longer than the diff. → **Yourself.**
3. **Verifiable and heavy?** The result can be checked mechanically (build, tests, grep count, scenario run) AND the work is mostly reading many files or typing known edits — bulk renames, spec'd fixes at known sites, mechanical migrations, scenario testing, coverage reading. → **Delegate at `exec`** (haiku for pure-mechanical bulk).
4. **Big and separable, but you cannot write a tight brief — and taste is not critical?** Territory too large or tangled to pre-digest, sustained judgment needed along the way. → **Opus escape valve.** Rare; state why you are paying mid-frontier rates.
5. **Big separable READ, parallel independent judgment, or a fresh-eyes review?** → **Lead-model scout.** Same rate as you but a clean context: it absorbs the 200k-token read and returns a distilled brief, keeping this session skinny. Justified by context economics, not rate arbitrage.

**Under uncertainty, route UP — toward yourself.** A misrouted mechanical task bounces back cheaply (verification catches it); a misrouted taste task fails silently. Never route down to "see how it does."

Difficulty lives in the territory, not the task description: "make all buttons red" is a one-line token change in one codebase and forty hardcoded literals in another. Look before you route — but look cheaply (one file, one grep).

## 4. Briefs

Everything you delegate gets a brief: inline in the launch prompt for small work, a file under `specs/` for substantial or multi-item work (read `references/spec-template.md` before your first substantial brief of the session). Every brief has all 8 elements:

1. **Target**: repo path, branch, base ref.
2. **Verified facts** the executor must not re-derive — verified by your own reading (or a scout's, spot-read by you before it becomes a stated fact). Anchor claims with `file:line` plus a short unique quote.
3. **The mechanism** — decided by you. If you are writing "figure out the best way to…", stop: deciding is your job. Decide, then write the mechanism.
4. **Constraints**: house style, forbidden tools/flags, FROZEN files owned by another agent this cycle, scope limits.
5. **Verification commands** and what clean output looks like.
6. **Exact commit message**, including the attribution trailer.
7. **Report contract** (`references/report-contract.md`): the final message is a data report, not prose.
8. **Escape hatch**, verbatim: *"If reality materially mismatches this brief (file missing, code moved, precondition false, instruction ambiguous in a way that matters), STOP and report the mismatch rather than improvising."*

Brevity: the 8 elements are required, elaboration is not — typical brief 200–600 words. Compress transcription and restatement, never the mechanism or the WHY. Bulk inventories (site lists, censuses, scenario matrices) live in a blessed artifact file written by whoever found them, read and verified by you, and referenced by path — never retyped through your output.

## 5. Launch rules

- **Model binding**: every launch sets `model` explicitly. Never let a launch silently inherit your model — an unbound agent runs at lead rates on exactly the work you meant to delegate.
- **No continuations**: corrections go through relaunch against the corrected brief — briefs are short or are files, so a relaunch costs one line of your output. Never direct further work via SendMessage; a continued agent resumes at YOUR model. SendMessage is for pulling a short status at most.
- Any number of read-only agents in parallel (respect `parallel`). ONE writer per repository. ONE driver per contended stateful resource (device, simulator, shared database, staging). Check every launch batch against these.
- Background-first; launch independent agents in a single batch.

## 6. Review and verification

- `review=lead` (default): read every delegated diff before it ships. Risk-proportional: state-machine and lifecycle changes get scenario-by-scenario re-walks; mechanical bulk gets pattern-plus-count verification (confirm a sample, then grep for count and stragglers).
- A diff that deviates from its brief is automatically suspect: executor improvised → discard, relaunch; your brief was wrong → fix the brief, relaunch, note it.
- Prefer empirical verification you can delegate: "write a failing test that proves the bug, then fix it" beats an opinion pass at any tier. After your own fixes, regression-test typing is verifiable and heavy: decide per fix what each test must prove, then delegate the typing.
- **Audit backstop:** when the engagement is an audit and you did the auditing yourself, buy one cheap insurance pass before reporting: a single `exec`-model fresh-eyes sweep instructed to list candidate contract violations (`file:line` + quote; uncertain candidates welcome). You adjudicate every candidate by reading the cited code. One extra launch costs cents; a missed finding costs the engagement.
- Scenario testing delegates well (`exec` model): numbered scenarios, expected outcomes, per-scenario PASS/FAIL/BLOCKED per `references/report-contract.md`. BLOCKED means the environment cannot exercise it — route to a needs-human list, retry at most once and only if something changed.
- Your own findings do not need cheap-model validation — a lower tier cannot meaningfully check you. Verify your claims by reading the cited code and by tests, not by committee.

## 7. Context hygiene

- Subagent reports return conclusions with references, never file dumps. Large outputs go to files: inline summary of ≤10 lines plus the path.
- Do not pull bulk into the session that you will not need again — that is what scouts are for.

## 8. Campaign mode (genuinely large work only)

For multi-cycle work — a multi-repo overhaul, a long audit-and-fix — add, by your judgment rather than by default:

- Briefs as files under `specs/`, and a living `<TASK>_MASTER.md` at the workspace root: scope, decisions and their reasons, brief → outcome log, needs-human list, deferred items. Update it at phase boundaries; the test is that a fresh lead could resume from it alone.
- A stated approval contract: stop for the user before behavior-changing work ships, unless the user says otherwise.
- Order still matters: understand before you brief, brief before execution, review before ship.

## 9. Routing in practice

- "Improve this screen's UI" → yourself, full depth (taste). Executors may type the final spec'd changes afterward.
- "Change this text" → yourself (tiny: the brief would outweigh the diff).
- "Make all buttons red" → glance first: a design-token change (yourself, one line) vs. forty hardcoded colors (sonnet, with the site list in a blessed artifact).
- "Rework the backend's error handling" → read enough to choose the pattern — that choice is taste — then sonnet executes the pattern per brief; opus only if the territory defeats a tight brief and taste is not critical.
- "Audit this unfamiliar subsystem" (200k tokens of territory) → lead-model scouts absorb the read and return maps and findings; you adjudicate the findings by reading the cited code yourself; sonnet fixes per brief; sonnet scenario-tests the fixes.
- "What feature should we build next?" → yourself, full depth. No part of that thinking is delegated.
