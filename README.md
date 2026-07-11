<p align="center">
  <img src="assets/banner.png" alt="/sub-minions" width="100%">
</p>

<h1 align="center">Same quality. Up to 42% less Fable.</h1>

<p align="center"><b>A routing discipline for your frontier lead:</b> design, judgment, and taste stay at full Fable depth; verifiable mechanical work runs on cheap Sonnet executors; small jobs it simply does itself. Measured at full quality parity in every regime tested.</p>

---

# sub-minions

## Why use it

- **Stretch your Fable quota.** On coverage-heavy work — audits, migrations, bulk refactors, scenario testing — 22–42% less frontier output at identical, independently-graded quality. The tokens that move to Sonnet are the reading and typing, never the thinking.
- **Never at taste's expense.** "Is this taste-critical?" is the *first* routing question: UI, architecture, naming, product feel, and every "what do you suggest?" get the lead's full depth, unabridged, no matter how small. Saving frontier tokens on copying is what pays for spending them freely on designing.
- **Safe to invoke at the start of any session.** The skill's resting state is vanilla: the lead behaves exactly as if the skill weren't loaded until a chunk of work passes an explicit launch justification test (name the mass moving off the frontier model AND the mechanical check that verifies it — or do it yourself). Its only unconditional cost is its own text (~2.5k cached tokens); everything else is conditional on a justified route, so the worst case is vanilla, not a process tax.
- **A quality mechanism vanilla doesn't have.** Every self-performed audit gets a cheap fresh-eyes backstop sweep; in testing it caught the one bug the lead missed, for cents.
- **Everything delegated is reviewed.** The lead reads every executor diff before it ships; deviations mean a corrected brief and a fresh executor, never silent drift.

## What it does

Invoked at session start (`/sub-minions`), the skill turns the lead into a router. Every task — or each separable chunk of one — gets one glance and one routing decision:

1. **Taste-critical?** (design, architecture, product feel, writing) → the lead does it itself, at full depth, always.
2. **Tiny?** (the instructions would be longer than the change) → the lead does it itself.
3. **Verifiable and heavy?** (checkable by build/tests/grep, and mostly reading many files or typing known edits) → delegated to a cheap executor against a tight 8-element brief: verified facts, the exact mechanism (decided by the lead), constraints, verification commands, a structured report contract, and a mandatory escape hatch — stop and report rather than improvise.
4. **Big but impossible to brief tightly, and not taste-critical?** → an Opus-tier escape valve, used rarely and stated explicitly.
5. **A huge separable read?** → a scout in a clean context that returns a distilled brief — but only when it *replaces* reading the lead would otherwise do.

Under uncertainty the router routes **up**, toward the lead: misrouted mechanical work bounces back cheaply off verification, misrouted taste work fails silently. And no launch happens without passing the justification test — one line naming the reading/typing mass that moves off the lead and the mechanical check that verifies the result; delegating because the option exists is a doctrine violation. Genuinely large multi-cycle work can opt into campaign mode — briefs as files, a living master doc a fresh session could resume from, and user checkpoints before behavior ships.

## Under the hood

- **Explicit model binding, no continuations** — every subagent launch names its model (an unbound launch silently inherits the lead's model and bills at frontier rates), and corrections always relaunch against a corrected brief rather than messaging a running agent (continuations resume at the lead's model).
- **Blessed artifacts, not retyping** — bulk inventories (site lists, censuses, scenario matrices) are written to spec-ready files by whoever found them, verified by the lead, and referenced by path, so transcription never flows through frontier output tokens.
- **Context hygiene** — large reports land as files with ten-line summaries, independent launches go out in single batches, and read-only helpers run in parallel with the lead's own work, because every extra turn re-reads the whole session at frontier cache rates.
- **Concurrency conventions** — any number of read-only agents in parallel, one writer per repository, one driver per stateful resource, frozen files named in every brief.

The skill is a directory: `SKILL.md` is the doctrine; `references/` holds the brief template and the report contracts.

## The evidence

Two data points shaped this design, one from Anthropic and one from our own head-to-head.

**At scale, delegation economics are real.** Anthropic measured a Fable 5 orchestrator with Sonnet 5 workers at **96% of solo Fable 5 performance for 46% of the price** on BrowseComp ([@ClaudeDevs](https://x.com/claudedevs/status/2074606063509528855)), and their [plan-big-execute-small cookbook](https://github.com/anthropics/claude-cookbooks/blob/main/managed_agents/CMA_plan_big_execute_small.ipynb) measured a coverage-research workload at **2.5× cheaper and 3× faster**, with 84% of input tokens billed at the cheap-worker rate. The win comes from fanning out ~900k tokens of *reading* at worker rates.

**Head-to-head measurements.** The skill was benchmarked against vanilla solo Fable on identical sealed fixtures with independent grading (seeded-bug answer keys, adversarial probes of delivered modules, verification run by the grader rather than self-reported):

| Workload | Quality (skill vs. solo) | Frontier output | Total cost |
|---|---|---|---|
| Small audit (375 lines, 10 seeded bugs) | 10/10 vs. 10/10 | **−13%** | +5–12% (within noise) |
| Large audit + 49-site migration (2k lines, 13 bugs) | 13/13 vs. 13/13 | **−22% to −42%** | −13% |
| Taste-led feature build (production Next.js app: scoring engine + API + UI) | parity (sealed 40-pt rubric; verified suites green in every run) | **+10%*** | single-run noise* |

<sub>*Taste-led work is the skill's null regime, and this row was measured twice. Before the launch-justification rule existed, the router launched 5 agents (4 unjustified orientation scouts) and measured +24% frontier output / +45% cost. Re-measured after: launches dropped to exactly 1 (a justified test-writer), frontier output narrowed to +10% of vanilla, and quality parity held. The residual output gap and the dollar spread sit within single-run variance — each run's exploration depth dominates cost at this task size; the delegation overhead itself is gone.</sub>

The doctrine follows the measurements. Delegation pays exactly where cheap reading and typing *substitute* for frontier work — the large-audit row. On taste-led work the router's correct move is to delegate almost nothing (the lead must read the territory to design against it), so the skill's job there is to stay out of the way and cost nothing. And the fresh-eyes backstop closed the only recall gap observed in any run (12/13 → 13/13) for cents.

Be honest with yourself about fit: **if your sessions are mostly small tasks and taste-led feature work, vanilla Fable needs no help** — this skill earns its keep on coverage-heavy work (big audits, migrations, bulk refactors, long test-driven sessions) and on quota-constrained days when Fable headroom is worth more than dollars.

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
