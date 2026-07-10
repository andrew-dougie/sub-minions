# conductor

An agent skill that codifies one orchestration pattern: **the lead model thinks, cheaper models execute.** A frontier-tier lead agent does all the hard work that actually requires a frontier model: scoping with the user, mapping unfamiliar code, adjudicating findings, writing implementation specs, and reviewing every diff. Disposable lower-tier executor agents do everything else: reading many files, making clearly-specified edits, running builds and test scenarios.

The pattern rests on two observations from real multi-repo working sessions. First, lead-model tokens are the scarce resource; a spec good enough that a mid-tier model executes it near-flawlessly is cheaper than having the frontier model type the edits itself, and far cheaper than reviewing a mid-tier model's improvisation. Second, unverified findings are expensive: audit agents produce confident false positives, so conductor layers verification: parallel audits, then an adversarial pass where fresh skeptic agents must CONFIRM or REFUTE each load-bearing claim with exact code quotes, then lead review of every diff, then independent end-to-end testing with strict PASS/FAIL/BLOCKED reporting. In the session this was distilled from, the adversarial pass alone killed 2 of 10 "P1" findings before they wasted an implementation cycle.

The skill is a directory: `SKILL.md` is the operating doctrine the lead adopts for the session; `references/` holds the copy-paste executor spec template, the report contracts, and the full verification ladder with collapse guidance for small tasks.

**Private repository. All rights reserved.** Not licensed for redistribution.

## Install and invoke

The repo root is the skill directory (`SKILL.md` at root), so installation is copying or symlinking this directory into the right place.

### Claude Code

```bash
git clone git@github.com:andrew-dougie/conductor.git
# global (all projects):
cp -R conductor ~/.claude/skills/conductor
# or per-project:
cp -R conductor <project>/.claude/skills/conductor
# symlink works too if the clone location is stable:
ln -s "$(pwd)/conductor" ~/.claude/skills/conductor
```

Invoke: `/conductor` at session start, with optional args:

```
/conductor exec=sonnet test=haiku checkpoints=every-cycle
```

Skills also auto-trigger by description, so asking for "orchestrated mode" or "lead + executor with subagents" can activate it without the slash command. Claude Code is the primary target: executor agents are real subagents with per-agent model selection and background execution.

### OpenAI Codex (CLI)

Codex supports the same skill format (a directory containing `SKILL.md`). Skills live in `$CODEX_HOME/skills/` (default `~/.codex/skills/`):

```bash
git clone git@github.com:andrew-dougie/conductor.git
cp -R conductor ~/.codex/skills/conductor
# restart Codex to pick up new skills
```

Fallback for older Codex versions: custom prompts (now deprecated by OpenAI in favor of skills, but still functional). Copy the `SKILL.md` body without the YAML frontmatter to `~/.codex/prompts/conductor.md`; it then appears in the `/` menu (as `/conductor` or `/prompts:conductor` depending on version). Only top-level `.md` files in that folder are scanned. For always-on doctrine instead of on-demand invocation, append the SKILL.md body to your project's `AGENTS.md`.

Args are passed as free text after the command (`/conductor exec=sonnet checkpoints=none`); the skill parses them from the invocation text, so no special argument plumbing is needed. Conventions verified against OpenAI's docs as of July 2026; if paths have moved, `~/.codex/skills/` (skill) and `AGENTS.md` (always-on) are the degrade-gracefully options.

### Cursor

Cursor commands are Markdown files invoked via `/` in chat:

```bash
git clone git@github.com:andrew-dougie/conductor.git
mkdir -p <project>/.cursor/commands
cp conductor/SKILL.md <project>/.cursor/commands/conductor.md
cp -R conductor/references <project>/.cursor/commands/conductor-references
# or globally for all projects:
cp conductor/SKILL.md ~/.cursor/commands/conductor.md
```

Invoke by typing `/conductor` in chat; args as free text after it. Update the `references/` paths mentioned in the command file if you relocate them (the lead can also just be told where they are). Alternative: install as a rule in `<project>/.cursor/rules/` and @-mention it manually. Conventions verified as of July 2026 (commands shipped in Cursor 1.6+).

## Args reference

Space-separated `key=value` pairs after the skill name. Unknown args are named and ignored; everything has a default.

| Arg | Values | Default | Meaning |
|---|---|---|---|
| `exec` | `haiku` \| `sonnet` \| `opus` | `sonnet` | Model for implementation executor agents |
| `test` | `haiku` \| `sonnet` \| `opus` | value of `exec` | Model for testing/verification agents |
| `verify` | `off` \| `spot` \| `adversarial` | `adversarial` for multi-finding audits, `spot` for small tasks | Audit-claim verification depth |
| `review` | `off` \| `lead` \| `independent` | `lead` | `lead` = orchestrator reads every diff; `independent` adds a separate review pass before merge-readiness |
| `checkpoints` | `none` \| `functional` \| `every-cycle` | `functional` | When to stop for user approval (`functional` = before functional/user-visible changes ship) |
| `parallel` | integer | `4` | Soft cap on concurrent background agents |
| `deploys` | `never` \| `ask` \| `safe` | `ask` | Whether agents may deploy (`safe` = only non-destructive, no schema/data impact) |
| `master-doc` | path \| `auto` \| `off` | `auto` | Living master doc (`auto` = `<TASK>_MASTER.md` at workspace root for multi-cycle work) |

On invocation the lead echoes the resolved configuration as a table and adopts it as standing session behavior.

## Anatomy of a good executor spec

The spec is why executors can be cheap: every design decision is made by the lead before delegation. Each spec carries eight elements: (1) exact repo path, branch, and base ref; (2) verified context stated as facts the executor never re-derives; (3) numbered work items, each with `file:line` anchors, the failure scenario it fixes, and the exact mechanism; (4) constraints including frozen files owned by other agents; (5) verification commands with expected clean output; (6) the full commit message including attribution trailer; (7) a structured report contract ("your final message is a data report, not prose"); (8) the escape hatch: stop and report on any material mismatch rather than improvising. That last line is the single most valuable one; it converts every would-be runaway into a cheap spec fix.

Copy-paste skeleton with inline guidance: [references/spec-template.md](references/spec-template.md). Report formats: [references/report-contract.md](references/report-contract.md). The full phase ladder with skip guidance: [references/verification-ladder.md](references/verification-ladder.md).

## Limitations

- **Platform asymmetry.** Only Claude Code has in-session subagents with per-agent model choice and background execution, so `exec`/`test`/`parallel` are only literal there. On Codex and Cursor the doctrine still applies, but "executors" are separate sessions or background agents fed spec files from a `specs/` directory, and the human schedules the parallelism.
- **Concurrency rules are convention, not enforcement.** One-writer-per-repo, one-driver-per-stateful-resource, and file freezes are honored because specs state them and the lead checks them at launch time. Nothing at the platform level prevents two agents from colliding if the lead schedules carelessly.
- **Verification depth costs real tokens.** Adversarial passes and independent reviews are extra agent runs. The defaults assume multi-cycle work where a false P1 is more expensive than a skeptic pass; for small tasks, collapse the ladder (the skill tells the lead how).
- **The lead is a single point of failure.** If the lead session dies, recovery depends entirely on master-doc discipline. That is why `master-doc=auto` is the default for multi-cycle work.
