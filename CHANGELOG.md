# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

**Version numbers matter for installation.** Claude Code only offers users an update when
the `version` field in `.claude-plugin/plugin.json` changes. Bump it — and the matching
`version` in `.claude-plugin/marketplace.json` — in the same commit as any change to the
skill or the subagents, or the fix reaches nobody.

## [Unreleased]

### Docs

- `HISTORY.md` and `HISTORY_ZH.md`: how the rules got this way, eight changes before 0.1.0,
  each with the problem, the change and the measured result where there is one. Both front
  pages link to it. A short version is the "Before 0.1.0" section at the end of this file.

## [0.1.1] — 2026-09-19

### Changed

- `skills/dirty-work/SKILL.md`: the `description` now says which kinds of work the skill
  covers (investigation, one-off rewrites that touch many places, migrations,
  self-contained builds, at any size) instead of listing phrases a user might type. A list
  of phrases is never complete. The body of the skill is unchanged.
  Measured cost and benefit, same smoke test as before: the new description alone fired on
  3 of 12 runs (the phrase list reached 8 of 12), and with the standing paragraph from the
  README it fired on 12 of 12, the same as before. False fires stayed at 0 of 8. The README
  now calls that paragraph required rather than recommended.
- `skills/dirty-work/references/codex.md`: the "Role definitions" section now points to
  the documented Codex custom-agent format (`~/.codex/agents/*.toml`) instead of saying the
  format was unverified. Still no Codex role files ship here.

### Docs

- README: numeric before/after table (illustrative numbers, labelled as such) next to a
  table of measured numbers, example models and list prices for each tier, and how to put
  your own models (DeepSeek and others) on the tiers.

## [0.1.0] — 2026-09-19

First public version. The rules themselves are not new: they are a cleaned-up, tool-neutral
version of a private skill the author has been using daily in Claude Code and Codex. What
is new is the packaging — the plugin manifests, the renamed subagents, and the English
text.

### Added

- `skills/dirty-work/SKILL.md` — the delegation rules: what must be delegated, what a work
  order must contain, how to behave while a subagent is running, verification, and the
  four-tier model selection with its escalation rule.
- `skills/dirty-work/references/claude-code.md` — Claude Code mechanics: where subagent
  files live, the frontmatter fields, and how to isolate parallel writes.
- `skills/dirty-work/references/codex.md` — Codex mechanics, including the two things you
  have to verify yourself about what a subagent inherits from its parent.
- `agents/` — seven ready-made Claude Code subagents: one per tier, plus a read-only
  auditor, a repair role and a read-only reviewer.
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` — Claude Code plugin
  and marketplace manifests.
- `README.md` and `README_ZH.md` — English and Chinese front pages.
- `AGENTS.md` and `CLAUDE.md` — instructions for an agent working in or installing from
  this repository. These are not shipped as context to users of the plugin.
- `CONTRIBUTING.md`, `SECURITY.md`, issue and pull request templates.
- `assets/banner.svg` — front-page banner.
- `LICENSE` — MIT.

### Known gaps in this release

Stated here rather than buried, because they are the reason this is 0.1.0 and not 1.0.0:

- **Nobody has installed this repository yet.** `/plugin marketplace add` and
  `npx skills add` are untested. The only check so far is a load test with
  `claude --plugin-dir`: the skill and the seven subagents register, and tier 1 and tier 4
  each answered one call.
- **There is no controlled comparison.** The numbers in the README are observational, from
  the author's own session logs with the private version of these rules.
- **The trigger test is a smoke test.** Small sample, one machine, one model, and the same
  requests were used to tune the `description`. Two kinds of request never fired.
- The Codex skills path convention is not fully settled; `references/codex.md` documents
  both readings and tells you how to check which one your version uses.
- `agents/tier4-frontier.md` ships with `model: fable`. Accounts without that model must
  edit one line — see the README.

### Notes

- No scripts ship with this skill, by design.
- The model column in the tier table is deliberately left for you to fill in. Tier
  definitions transfer between tools; model names go stale in months.

## Before 0.1.0

The rules were a private skill for about ten weeks before this repository existed. These
are the changes I can back with a file. Each line: date, the problem, the change, the
result. Numbers come from my own session logs (observational, 2026-09-05 to 09-19).

- **2026-07-10 to 07-14** — 20+ subagents (then 12 more) silently inherited the main
  session's most expensive model. Every dispatch now names its model; shipped role files
  pin theirs. Result: 2 of 123 later dispatches (1.6%) ran on the top model.
- **2026-08-03 to 08-09** — Routine tasks were escalated to the top model, a reviewer's
  rejection was booked as a failed delegation, and a 10-line reply cap squeezed out
  needed information. Escalation now counts task reports, a rejection is a successful
  review, and replies have no fixed length. Result: the median report is still only 272
  tokens.
- **2026-08-11 to 09-02** — Work-order files, per-dispatch logs, hooks and automatic
  repair tickets turned small infrastructure gaps into endless loops. All of it was
  retired; no fixed work-order form, no scripts. Result: a dispatch is one call, about
  365 tokens of work order.
- **2026-09-02** — An agent judged a diagnosis "small", did it itself, used about 200K of
  a 258K context in one turn and lost its findings to compaction. Independent work now
  goes to a subagent by default. Result: not enough, see the next line.
- **2026-09-03** — A four-hour task made zero delegations; the main conversation grew
  from about 65K to about 290K tokens. The skill had never been read. The trigger moved
  into always-loaded text. Result: trigger test 3 of 12 without the standing paragraph,
  12 of 12 with it; in later delegating sessions 88.2% of reads and commands ran in
  subagents, and 0.21% of the material they handled came back.
- **2026-09-03 to 09-06** — Preparing a work order turned into doing the investigation,
  and an unverified inference went into a work order as fact. Investigation now goes out
  with its open questions attached. Result: one example only (main agent read 3 files,
  24 subagents made about 2,800 tool calls).
- **2026-09-15 to 09-18** — A two-day job used about 1.2 billion tokens: 63.7% on
  item-by-item verdicts at about 24,200 tokens each, 20.8% on waiting and polling. Added:
  follow up only when needed, measure per-item cost on a small batch first, re-tier as
  you go. Result: no before-and-after number yet.
- **2026-09-06 to 09-18** — One strong model covered two tiers and took 78% of dispatches.
  Tiering became one question (how much must the subagent decide for itself?) with one
  model per tier. Result: no after-number yet; 78% is the baseline.

The full story, with evidence and with what was never measured, is in
[HISTORY.md](HISTORY.md).

[Unreleased]: https://github.com/humeicw/dirty-work/compare/v0.1.1...HEAD
[0.1.1]: https://github.com/humeicw/dirty-work/releases/tag/v0.1.1
[0.1.0]: https://github.com/humeicw/dirty-work/releases/tag/v0.1.0
