![dirty-work: the main agent sends investigation and bulk reading to tier 1 to tier 4 subagents, which return conclusions instead of file dumps](assets/banner.svg)

# dirty-work

[中文说明 / Chinese README](README_ZH.md)

**Your agent has subagents. It still reads everything itself.**

A skill that makes the main agent send investigation and bulk reading to subagents, take
back only the conclusions its current task needs instead of a pile of raw material, and
route each job to the smallest model tier that can do it well. Less context rot, fewer
tokens burned on your most expensive model, and the order is fixed: get the job right
first, keep the context clean second, save tokens third. Let the minions do the dirty work.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Works with](https://img.shields.io/badge/works%20with-Claude%20Code%20%7C%20Codex-black)](#what-has-been-tested-and-what-it-does-not-do)
[![Format](https://img.shields.io/badge/format-Agent%20Skills-6f42c1)](#install)
[![Scripts](https://img.shields.io/badge/scripts-none-brightgreen)](#faq)
[![Version](https://img.shields.io/badge/version-0.1.1-lightgrey)](CHANGELOG.md)

```bash
npx skills add humeicw/dirty-work
```

## Before and after

Same request, one hour in: *"figure out why the build broke after last week's merge."*

| After one hour | Without this skill | With dirty-work |
|---|---|---|
| How it works the problem | Greps, opens files and pulls build logs itself. All of it lands in the main conversation. | Sends one subagent to dig. Back come the root cause and the few lines that matter. |
| Tokens in the main conversation | **~180k** | **~60k** |
| Context window used | **90%** - *"context low, compacting"* | **30%** |
| Share of that context that is about your question | **~15%** | **~85%** |
| Cost of reading 1M tokens of logs and files | **$10** on the top model | **$1 to $2** on a tier-1 or tier-2 model |

*Illustrative round numbers for a typical debugging session in a 200k-token window. They
show the shape and are not a benchmark. Prices are list input prices on 2026-09-19.*

What my own logs show, over 14 days of real work:

| Measured | |
|---|---|
| Material read by subagents | ~19.0M tokens |
| What came back into the main conversation | ~40.9k tokens, which is 0.21% |
| Median size of what one delegation brought back | 272 tokens |
| File reads and shell commands that ran inside subagents | 88% |
| The one session where nothing was delegated (2026-09-03, about 4 hours) | main conversation grew from 65k to 290k tokens |

How these were counted, and what they do not show: [What I measured](#what-i-measured).

## How it works

**1. Investigation and bulk reading go out.** Anything where the agent would have to hunt
(which file is authoritative, what did we miss) is delegated with the open questions
attached. Large bodies of material go to a reader subagent. The main agent reads the entry
point and the state file, not the pile, and may not investigate *to write the work order*
either.

**2. Subagents return conclusions, not material.** The work order asks for the answer and
the evidence that matters, not the source text. While a subagent runs, the main agent may
not repeat that search or re-read the source to "check" the report.

### The four tiers

One question decides the tier: **how much of this job is not written down in the work order
and has to be decided by the subagent?** Check 4, then 3, then 2, then 1, and stop at the
first hit.

| Tier | The subagent has to... | Typical jobs |
|---|---|---|
| **1 Mechanical** | ...follow a stated rule. Input, rule and output are written down, and the result is checkable. | Replace terms from a list, convert formats, run a given command, apply a given diff. |
| **2 Understanding** | ...understand the content to judge each item, with no overall design to hold. | Read material and report what matters, classify, edit within named files. |
| **3 Reasoning** | ...hold one line of reasoning across interlocking steps. Approach and acceptance are fixed. | Implement an agreed design across modules, reason from evidence to a root cause. |
| **4 Exploration** | ...find the path itself, or make a call that is hard to undo. | Debug a failure with no known source, set shared rules, decisions touching production. |

Once tier 4 has settled the path, the rest drops to tier 3, scoped edits to tier 2,
mechanical work to tier 1.

### Which model for each tier

What I run today, with list prices per 1M tokens (input / output) on 2026-09-19:

| Tier | Claude Code | Price | Codex | Price |
|---|---|---|---|---|
| 1 | Haiku 4.5 | $1 / $5 | gpt-5.6-luna | $0.20 / $1.20 |
| 2 | Sonnet 5 | $2 / $10 | gpt-5.6-terra | $2 / $12 |
| 3 | Opus 5 | $5 / $25 | gpt-5.6-sol | $4 / $20 |
| 4 | Fable 5.1 | $10 / $50 | gpt-6-astra | $10 / $50 |

Tier 1 costs a tenth of tier 4 on Claude and a fiftieth on OpenAI.

**Use your own models.** A tier is a definition, not a brand, and model names go stale in
months. The skill's copy of the tier table has a model column you fill in once. In Claude
Code each file in `agents/` has one `model:` line that takes an alias, `inherit` or a full
model ID, and the `ANTHROPIC_DEFAULT_HAIKU_MODEL`, `..._SONNET_MODEL` and `..._OPUS_MODEL`
environment variables point the aliases at something else. In Codex, set `model` in each
`~/.codex/agents/*.toml` and add other providers under `[model_providers]` in
`config.toml`. DeepSeek, for one, has an Anthropic-compatible endpoint, so `deepseek-flash`
can take tiers 1 and 2 and `deepseek-v4-pro` tiers 3 and 4. I have only run Claude and
OpenAI models myself. One rule: tiers 3 and 4 must not end up on the same model at the same
effort, or four tiers collapse into three.

## Install

Claude Code, as a plugin. The only route that also installs the seven tier subagents.

```
/plugin marketplace add humeicw/dirty-work
/plugin install dirty-work@dirty-work
```

The skill is then `/dirty-work:dirty-work`, and the subagents appear in `/context`.

> `agents/tier4-frontier.md` ships with `model: fable`. Without access to that model,
> change `model:` to the strongest one you have and keep `effort: xhigh`, or every tier-4
> call fails with a model error.

<details>
<summary>Other tools, by hand, and Claude Desktop</summary>

Codex, Cursor, Gemini CLI, Copilot, OpenCode and others, via the cross-tool CLI:

```bash
npx skills add humeicw/dirty-work
```

By hand: copy the self-contained `skills/dirty-work/` into `~/.claude/skills/`,
`~/.codex/skills/`, or your tool's project skills directory. For the subagents, copy
`agents/*.md` into `~/.claude/agents/`.

Claude Desktop or claude.ai: zip `skills/dirty-work/` and upload it as a skill. This
carries the skill only, not the subagents.

Uninstalling: see [AGENTS.md](AGENTS.md).

</details>

## Making it trigger

**This step is required, not optional.** Installing a skill does not make the model load
it: agents consult skills when a task looks beyond them, and *"let me just check one file"*
never does. Add this to your `CLAUDE.md` or `AGENTS.md`:

```
You must read the dirty-work skill and follow it before your first file read, grep or shell
command on any task that means investigating or debugging something, searching the
codebase, reading a lot of files or logs, a repo-wide rewrite or migration, or building
something across several files. This applies even when the task looks small. Outside that
list, do not read it: not for a single named edit, not for a question you can answer from
what is already in front of you, and not for a document you and the user are writing
together turn by turn.
```

The point is the *before*. Once the agent has read five files, the context is already
spent.

In a [smoke test](evals/trigger-smoke-test.md) (14-file test repository, Claude Code
2.1.258, Sonnet, 6 requests the skill should catch, 2 runs each), it fired on 3 of 12 runs
with the description alone and 12 of 12 with this paragraph. On 4 requests it should
ignore, it fired on 0 of 8 runs either way. Those requests were also used while choosing
the wording, so a second round used 10 new requests that were never used for tuning: 0 of
12 with the description alone, 12 of 12 with the paragraph, and 0 of 8 false fires either
way. The description says which kinds of work the skill covers instead of listing phrases a
user might type. An earlier phrase-list version reached 8 of 12 on its own in the first
round but 0 of 12 on the new requests, and two other rewordings also scored 0 of 12 there.
Rewording the description did not help; the paragraph is what made the difference. With
the paragraph, both the phrase list and the current description reach 12 of 12 in the
first round.
Twelve of twelve is not "always": small sample, one machine, one model, one run per
condition, and the paragraph was injected with `--append-system-prompt` rather than a real
`CLAUDE.md` in all but 3 spot checks.

## Why I made this

On 2026-09-03 I gave my agent a large reorganization job. It ran for about four hours and
never once delegated. Every search, every file and all 61 shell commands landed in the main
conversation, which grew from 65k to 290k tokens. Subagents were available the whole time,
but each step looked small. I am not a
professional developer; I use Claude Code and Codex every day for my own work, and wrote
these rules after watching this repeat.

Long context is not free, and the damage starts before you hit the limit:

- Anthropic's subagent documentation lists **"Preserve context"** as the first benefit of
  subagents. ([Claude Code docs](https://code.claude.com/docs/en/sub-agents))
- Chroma's **Context Rot** report held difficulty fixed across 18 frontier models and only
  lengthened the input; reliability fell well before any limit.
  ([Chroma Research, 2025](https://research.trychroma.com/context-rot))
- **Lost in the Middle** (Liu et al., [arXiv:2307.03172](https://arxiv.org/abs/2307.03172))
  found information buried mid-context is used much less reliably than at either end.
- Anthropic's **Effective context engineering for AI agents** frames context as an
  *attention budget*.
  ([Anthropic Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents))

Those are other people's findings, not mine. The second reason is usage: on a subscription
the scarce thing is your rate-limit window, and a tier-1 job on a frontier model burns the
same window as work you care about.

What went wrong before each rule was added, and what changed after, is in
[How these rules got this way](HISTORY.md).

## What I measured

Fourteen days of my own Claude Code logs (2026-09-05 to 09-19), from the private version of
these rules: 19 sessions that delegated, 114 dispatches plus 49 follow-ups.

The numbers are in the second table under [Before and after](#before-and-after). Tier mix:
78% tier 3, 20% tiers 1 and 2, 2% tier 4. My work is mostly multi-step, so the cheap tiers
are the minority here.

This is observational data from real, differing tasks, not a controlled comparison: it shows
how much stayed out of the main context, not how much better the results were. Token counts
are estimates, counted once per piece of material, not per cache re-read.

## What has been tested, and what it does not do

The rules have daily use behind them, in Claude Code and Codex, from a private version of
this skill. **The packaging has had less testing**: on 2026-09-23, on one Windows machine
with a fresh, empty config, both install routes worked. `/plugin marketplace add` plus
`/plugin install` (Claude Code 2.1.258) installed the skill and all seven agents;
`npx skills add` (skills 1.7.0, targeting Claude Code) installed the complete skill and
parsed its description. Earlier, loaded with `claude --plugin-dir`, tier 1 and tier 4 each
answered one call. Install-then-invoke in one real session is not tested yet. Other Agent
Skills tools should work, since the skill is plain text, but I have not tried them.

- The benefit is cumulative: you notice it three hours into a session that did not need
  compacting.
- Delegation is not free: every subagent re-reads its background, so on a small task doing
  it yourself is cheaper. That is my judgment, not a rule in `SKILL.md`, which delegates
  independent work by default.
- Tier 1 and 2 subagents sometimes answer worse than the main agent would have. Section 7
  escalates, at the cost of a round trip.
- Trigger reliability is the weak point of every skill, mine included. Without the paragraph
  from [Making it trigger](#making-it-trigger), the skill loaded on only 3 of the 24 runs
  that should have loaded it, across both rounds of the
  [smoke test](evals/trigger-smoke-test.md).

## How this differs from other delegation skills

The similar projects I know of fall into three groups; any may suit you better.

- **Model routers** send a job to a cheaper model; they do not ask whether the main agent
  should be doing it at all.
- **Subagent personas plus an orchestration runtime** answer a different question: what
  should the helper look like.
- **"When to delegate" guidance** is the closest neighbor, usually a short heuristic rather
  than a full procedure.

This is a decision rule for the **main** agent: what a work order must contain and that it
can never widen the subagent's authority, when to re-tier, escalation after two failures at
the same tier, and one owner per piece of work. It ships no scripts.

## FAQ

**The name sounds like you are dumping unpleasant work on someone.**
That is the joke, and it is only half a joke. The reading and the searching genuinely go
out. Responsibility does not: the main agent writes the work order, judges the answer,
escalates when it is not good enough, and answers to you for the result.

**Doesn't delegating use more tokens, not fewer?**
In total, often yes: every subagent re-reads its background. What goes down is tokens in
the main thread, the one that has to stay coherent for hours, and tokens on your most
expensive model: reading a million tokens of logs costs $10 on my tier-4 model and $1 on
tier 1. Whether the bill drops depends on your mix. Most of my own work is tier 3, so what
I count on is a cleaner context, not a smaller bill.

**My model has a huge context window and my tool compacts automatically. Why isn't that
enough?**
A bigger window postpones compaction. On the evidence above it does not remove the
reliability decay that comes with longer input, and a compacted session has already lost
detail. That is my reading of those papers, not a measurement.

## Contributing and license

Issues help more than pull requests right now, especially a case where the skill should
have triggered and did not: include the exact prompt. See
[CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md).

MIT. Changes are in [CHANGELOG.md](CHANGELOG.md).

---

If this saved you one compaction, a star helps the next person find it. If it did not, open
an issue: that is worth more to me than a star.
