<p align="center">
  <img src="assets/banner.png" alt="/sub-minions" width="100%">
</p>

<h1 align="center">Spend Fable where it matters</h1>

<p align="center"><b>A frontier-token routing discipline.</b> Taste and judgment stay with the frontier lead at full depth; verifiable mechanical work runs on cheap executors; below the delegation floor the lead just does the work itself.</p>

---

# sub-minions

An agent skill that codifies one discipline: **route every task by taste, verifiability, and reading mass — then protect the routing with review.** The frontier lead (Fable) keeps everything where quality depends on judgment no mechanical check can score: UI and UX, architecture, API shape, naming, product feel, "what should we build?" Work that is mechanically verifiable and heavy in reading or typing — bulk edits, spec'd fixes, migrations, scenario testing, coverage reading — is delegated to cheap executor subagents against tight briefs. Tiny tasks the lead just does: below the delegation floor, a brief costs more than the diff.

There is deliberately no standing machinery — no phase ladder, no always-on orchestration mode. Routing is a per-task reflex, and "the lead does it itself" is a first-class outcome. The design is grounded in measurement (see the evidence section): delegation pays off in proportion to how much cheap reading and typing it moves off the frontier model, and orchestration run for its own sake costs more than it saves.

## Features

- **A five-way routing table, applied as a reflex** — taste-critical → lead at full depth, always; tiny → lead (brief would outweigh the diff); verifiable + heavy → cheap executor; big-but-unbriefable and not taste-critical → Opus escape valve; big separable reads → lead-model *scouts* that absorb huge reading in a clean context and return distilled briefs (justified by context economics, not rate). Under uncertainty, route up: misrouted mechanical work bounces back cheaply, misrouted taste work fails silently.
- **Judgment stays frontier, structurally** — taste never routes down because taste is the first routing criterion, not a guardrail bolted on. The savings exist so the lead can think, design, and delight without rationing.
- **8-element briefs** — verified facts (anchored `file:line` + quote), the decided mechanism, frozen-file constraints, exact verification commands, the commit message, a structured report contract, and a mandatory escape hatch ("STOP and report rather than improvising"). Typical brief: 200–600 words — compress transcription, never the mechanism or the why.
- **Blessed artifacts, not retyping** — bulk inventories (site lists, censuses) are written to spec-ready files by whoever found them, verified by the lead, and referenced by path, so transcription never flows through frontier output tokens.
- **Explicit model binding, no continuations** — every launch names its model (unbound subagents silently inherit the lead's model and bill at frontier rates); corrections relaunch against the corrected brief because messaged agents resume at the lead's model (observed in the field).
- **Review as the quality floor** — the lead reads every delegated diff (input tokens: the cheap direction); mechanical bulk gets pattern-plus-count verification; empirical checks ("write the failing test first") beat opinion passes at any tier.
- **Context hygiene** — large reports land as files with ten-line summaries; the resident session stays skinny because everything in it is re-read on every later turn.
- **Campaign mode, opt-in by judgment** — genuinely large multi-cycle work adds briefs-as-files under `specs/`, a living master doc a fresh lead could resume from, and user checkpoints before behavior ships. It is a paragraph of discipline, not a process.

The skill is a directory: `SKILL.md` is the doctrine; `references/` holds the brief template and the report contracts.

## The evidence

Two data points shaped this design, one from Anthropic and one from our own head-to-head.

**At scale, delegation economics are real.** Anthropic measured a Fable 5 orchestrator with Sonnet 5 workers at **96% of solo Fable 5 performance for 46% of the price** on BrowseComp ([@ClaudeDevs](https://x.com/claudedevs/status/2074606063509528855)), and their [plan-big-execute-small cookbook](https://github.com/anthropics/claude-cookbooks/blob/main/managed_agents/CMA_plan_big_execute_small.ipynb) measured a coverage-research workload at **2.5× cheaper and 3× faster**, with 84% of input tokens billed at the cheap-worker rate. The win comes from fanning out ~900k tokens of *reading* at worker rates.

**Our own head-to-head measurements.** We ran this skill against vanilla solo Fable on identical sealed fixtures with independent grading (seeded-bug answer keys, adversarial probes of delivered modules, verification run by the grader rather than self-reported):

| Workload | Quality (skill vs. solo) | Frontier output | Total cost |
|---|---|---|---|
| Small audit (375 lines, 10 seeded bugs) | 10/10 vs. 10/10 | **−13%** | +5–12% (within noise) |
| Large audit + 49-site migration (2k lines, 13 bugs) | 13/13 vs. 13/13 | **−22% to −42%** | −13% at best config |
| Taste-led feature build (production Next.js app, scoring engine + API + UI) | parity (38 vs. 37.5 on a sealed 40-pt rubric) | **+24%** | +45% |

Three lessons are encoded in the doctrine. Routing beats standing orchestration: an always-on pipeline (mappers, skeptics, executor fan-out) over the small fixture cost 2.6× solo at identical quality. Delegation pays exactly where cheap reading/typing *substitutes* for frontier work — the large-audit rows. And on taste-led work the correct answer is near-zero delegation: scouts that precede rather than replace the lead's own reading are pure overhead (the +24% row — its causes, orientation scouts and extra delegation rounds, are now gated in the doctrine). A cheap fresh-eyes backstop closed the one recall gap ever observed (12/13 → 13/13) for cents.

No formal license; shared publicly as-is.

## Install and invoke

This skill is built for Claude Code — executors are real in-session subagents with per-agent model selection and background execution, which is what routing depends on. The repo root is the skill directory (`SKILL.md` at root), so installation is copying or symlinking this directory into the right place:

```bash
git clone https://github.com/andrew-dougie/sub-minions.git
# global (all projects):
cp -R sub-minions ~/.claude/skills/sub-minions
# or per-project:
cp -R sub-minions <project>/.claude/skills/sub-minions
# symlink works too if the clone location is stable:
ln -s "$(pwd)/sub-minions" ~/.claude/skills/sub-minions
```

Invoke: `/sub-minions` at session start, with optional args:

```
/sub-minions exec=sonnet review=lead
```

Skills also auto-trigger by description, so asking to "save Fable tokens" or "delegate mechanical work to cheaper models" can activate it without the slash command.

## Args reference

Space-separated `key=value` pairs after the skill name. Unknown args are named and ignored; everything has a default.

| Arg | Values | Default | Meaning |
|---|---|---|---|
| `exec` | `haiku` \| `sonnet` \| `opus` | `sonnet` | Default model for delegated work; the lead may downgrade a pure-mechanical brief to haiku |
| `review` | `lead` \| `off` | `lead` | `lead` = the lead reads every delegated diff before it ships |
| `parallel` | integer | `4` | Soft cap on concurrent subagents |
| `deploys` | `never` \| `ask` \| `safe` | `ask` | Whether agents may deploy (`safe` = only non-destructive, no schema/data impact) |

## Limitations

- **Claude Code only.** Routing depends on in-session subagents with per-agent model choice and background execution; no other agent platform currently provides them.
- **Routing is judgment, not enforcement.** The table tells the lead how to route; nothing stops a bad glance from misclassifying a task. The route-up-under-uncertainty rule and mandatory diff review are the containment, and they trade some savings for quality by design.
- **Agent continuations inherit the lead's model.** In Claude Code, messaging a subagent (SendMessage) can resume it at the session model rather than its launched model — observed in practice silently turning Sonnet executors into frontier-priced ones mid-run. The skill therefore corrects executors by relaunching against the brief, never by continuation.
- **Concurrency rules are convention, not enforcement.** One-writer-per-repo, one-driver-per-stateful-resource, and file freezes are honored because briefs state them and the lead checks them at launch time.
- **Savings scale with the work's shape.** Coverage-heavy sessions can push most tokens to cheap tiers; pure-taste sessions save ~0% — which is the design working as intended, not failing.

Issues and suggestions welcome.
