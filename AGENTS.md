# dirty-work — install and use

Instructions for an agent working in this repository, or for an agent asked to install this
skill for its user.

## What is here

```
skills/dirty-work/
  SKILL.md                    the rules — this is the whole skill
  references/claude-code.md   Claude Code mechanics
  references/codex.md         Codex mechanics
agents/                       seven ready-made Claude Code subagents
.claude-plugin/               plugin + marketplace manifests
.github/                      issue forms and the pull request template
assets/banner.svg             front-page banner
README.md, README_ZH.md       front pages, English and Chinese
CONTRIBUTING.md               what helps, what is declined, the pre-PR checklist
SECURITY.md                   what ships, and how to report a problem
CHANGELOG.md                  Keep a Changelog; version bumps are what trigger updates
LICENSE                       MIT
```

Only `skills/` and `agents/` reach a user's machine. Everything else exists for people
working on this repository.

There is no `SKILL.md` at the repository root, and there must never be one: `npx skills add`
stops at a root `SKILL.md` and would then ignore everything under `skills/`.

This file and the `CLAUDE.md` next to it are for people and agents working *in this
repository*. They are not shipped as context: installing the plugin does not load them into
anyone's session. What installing gives you is the skill under `skills/` and the subagents
under `agents/`.

No scripts ship here. Nothing needs to be executed to install this.

## Installing

**Claude Code, as a plugin.** This is the only route that also installs the seven tier
subagents.

```
/plugin marketplace add humeicw/dirty-work
/plugin install dirty-work@dirty-work
```

Plugin skills are namespaced, so the skill is `/dirty-work:dirty-work`. The seven
agents show up in `/context` under Custom Agents.

**Any other tool**, via the cross-tool CLI:

```bash
npx skills add humeicw/dirty-work
```

Add `-g` to install globally instead of into the current project. Add `-a claude-code` or
`-a codex` to target a specific tool.

**By hand.** Copy the directory `skills/dirty-work/` — the whole directory, it is
self-contained — into one of:

- Claude Code, personal: `~/.claude/skills/`
- Claude Code, project: `.claude/skills/`
- Codex, personal: `~/.codex/skills/`
- Codex, project: `.codex/skills/` (see `references/codex.md`; the convention is not fully
  settled)

If you install by hand and want the subagents too, copy `agents/*.md` into
`~/.claude/agents/`.

## After installing: two things to set up

**1. Make it trigger.** Installing does not guarantee the skill loads. Add this paragraph to
the user's `CLAUDE.md` or `AGENTS.md`:

```
You must read the dirty-work skill and follow it before your first file read, grep or shell
command on any task that means investigating or debugging something, searching the
codebase, reading a lot of files or logs, a repo-wide rewrite or migration, or building
something across several files. This applies even when the task looks small. Outside that
list, do not read it: not for a single named edit, not for a question you can answer from
what is already in front of you, and not for a document you and the user are writing
together turn by turn.
```

**2. Fill in the model column, and check tier 4.** Section 8 of `SKILL.md` has a tier table
with a column marked "Your model (fill this in)". Ask the user which models they have access
to and write one model per tier into their copy, cheapest first. The tier definitions are
what transfers between tools; model names are not.

`agents/tier4-frontier.md` ships with `model: fable` and `effort: xhigh`. **If the user does
not have access to that model, change that line to the strongest model they do have** —
`opus` with `effort: xhigh` is the usual fallback. Tier 3 and tier 4 must not end up on the
same model at the same effort, or the four-tier table collapses into three.

## Uninstalling

- Plugin: open the `/plugin` manager and remove `dirty-work`.
- CLI: `npx skills remove dirty-work` (the command takes the skill name, not
  `owner/repo`).
- By hand: delete the `dirty-work` directory from the skills path, and any
  `agents/tier*.md` files you copied.
- Also remove the paragraph added to `CLAUDE.md` or `AGENTS.md`.

## Editing this repository

- `skills/dirty-work/SKILL.md` must stay under 500 lines and roughly 5,000 tokens. The
  spec loads the whole body once the skill triggers; anything longer belongs in
  `references/`.
- The `name` in the skill's frontmatter must equal its directory name, and the frontmatter
  carries `name` and `description` only.
- The `description` must never contain a colon followed by a space. Claude Code tolerates
  it, but standard YAML parsers reject it, so `npx skills add` and other tools would fail.
- Any change to the skill needs a matching `version` bump in `.claude-plugin/plugin.json`
  **and** `.claude-plugin/marketplace.json`, plus an entry in `CHANGELOG.md`. Users are only
  offered an update when `version` changes.
- Tool-specific mechanics go in `references/`, never in the body of `SKILL.md`.
- Run `claude plugin validate ./` before publishing a change. Add `--strict` to treat
  warnings as errors.
- Keep `README.md` and `README_ZH.md` in step: same sections, same order, same placeholders.
- Do not claim in the README that something has been tested until it actually has.
- `CONTRIBUTING.md` has the rules for contributors. Read it before proposing a change on
  someone's behalf.
