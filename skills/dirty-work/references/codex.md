# Codex mechanics

Everything in `SKILL.md` is tool-independent. This file is only the Codex plumbing.

**Status of this file:** the notes below come from the author's own day-to-day Codex setup,
not from an official Codex reference page. Treat them as a starting point and confirm them
against your own version before relying on them. Where the ecosystem disagrees, that is
said explicitly.

Last checked: 2026-09-18. The "Role definitions" section was rechecked on 2026-09-19.

## Where Codex reads skills from

Two conventions are in circulation and they do not fully agree:

- Most third-party guides say Codex reads `~/.codex/skills/` (personal) and
  `.codex/skills/` (project).
- The `skills` CLI (`npx skills add`) installs to `~/.codex/skills/` globally but to
  `.agents/skills/` for a project.

`.agents/skills/` is the newer cross-tool path. Whether Codex reads both has not been
confirmed against official documentation. **Check which path your version actually picks up
before assuming the skill is loaded** — the simplest test is to install it and ask Codex to
name the skills it can see.

## Role definitions

This repository ships no Codex role files. The official Codex documentation
(developers.openai.com/codex/subagents) describes custom agents as TOML files in
`~/.codex/agents/` (personal) or `.codex/agents/` (project), with the fields `name`,
`description`, `developer_instructions`, `model` and `model_reasoning_effort`. Setting
`default_subagent_model` under `[agents]` in `config.toml` gives all of them a default. I
use that format in my own setup, but I have not packaged and tested role files for this
repository, so build your own, one per tier, and check the field names against the
documentation for your Codex version.

For the instruction text, take the Claude Code role files from the repository's `agents/`
directory and drop the frontmatter. The instructions are portable between tools; the model
line is not. The seven roles and which tier each one belongs to are listed in
`claude-code.md`.

## Do not let a subagent inherit the whole conversation

This is the Codex-specific detail that matters most for context hygiene, and it is worth
checking on your own version before you rely on tier routing at all.

Find out what your Codex version does by default when you spawn a subagent:

- Does the subagent start with a clean context, or does it inherit the parent
  conversation's history?
- Does it start on the model and reasoning effort you asked for, or does it inherit the
  parent's?

In the author's setup, the default is to inherit both, and the inherited model cannot be
overridden at the same time. If that is also true for you, a "cheap tier-1 subagent" will
silently run on the parent's expensive model with the parent's entire history loaded — the
exact opposite of what this skill is for. Turn inheritance off explicitly for independent
tasks, and pass only a limited slice of history when the subagent genuinely needs recent
context.

The simplest test: spawn a subagent for a trivial job, ask it what it knows about the
conversation so far, and check which model it reports running on.

## When you are not using a role file

Specify the model and reasoning effort explicitly on the call, and restate the authority
boundary in the work order. Dropping the role file does not drop the boundary.
