# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

**Version numbers matter for installation.** Claude Code only offers users an update when
the `version` field in `.claude-plugin/plugin.json` changes. Bump it — and the matching
`version` in `.claude-plugin/marketplace.json` — in the same commit as any change to the
skill or the subagents, or the fix reaches nobody.

## [Unreleased]

Nothing yet.

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

[Unreleased]: https://github.com/humeicw/dirty-work/compare/v0.1.1...HEAD
[0.1.1]: https://github.com/humeicw/dirty-work/releases/tag/v0.1.1
[0.1.0]: https://github.com/humeicw/dirty-work/releases/tag/v0.1.0
