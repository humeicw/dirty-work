# Claude Code mechanics

Everything in `SKILL.md` is tool-independent. This file is only the Claude Code plumbing.

Last checked against the official docs: 2026-09-18.
Claude Code changes often — re-check the pages linked at the bottom before relying on any
detail here.

## Where subagent definitions live

- Personal: `~/.claude/agents/<name>.md`
- Project: `.claude/agents/<name>.md`
- Shipped by a plugin: `<plugin>/agents/<name>.md` — auto-discovered, no manifest entry
  needed

The dirty-work repository ships seven ready-made roles in its `agents/` directory. If
you installed it as a plugin, they are registered for you. If you only copied the skill
folder, fetch `agents/*.md` from the repository and put them in `~/.claude/agents/`.

## The seven roles

| Tier | Agent name | Model / effort as shipped | What it is for |
|---|---|---|---|
| 1 | `tier1-mechanical` | `haiku`, `maxTurns: 40` | Stated rule in, checkable result out. |
| 2 | `tier2-routine` | `sonnet`, `medium` | The default for reading material and reporting back. |
| 2 | `tier2-auditor` | `sonnet`, `medium`, read-only | Item-by-item check against fixed rules. Reports, never fixes. |
| 2 | `tier2-iteration` | `sonnet`, `medium` | Minimal fix when root cause and remedy are already known. |
| 3 | `tier3-complex` | `opus`, `high` | Interlocking steps inside a fixed approach. |
| 3 | `tier3-reviewer` | `opus`, `high`, no Write/Edit | Risks, counterexamples and omissions in an implementation or plan. |
| 4 | `tier4-frontier` | `fable`, `xhigh` | Unknown path, or a decision that is hard to undo. |

Name the role you want when you delegate. `tier2-routine` and `tier3-complex` carry most of
the traffic; the other five are called by name when the job matches them.

**Check tier 4 before you rely on it.** It ships as `fable` with `effort: xhigh` so that it
is not the same configuration as tier 3. If your account does not have that model, edit
`agents/tier4-frontier.md` to name the strongest model you do have — `opus` with
`effort: xhigh` is the usual fallback. Tier 3 and tier 4 must never end up identical, or
the four-tier table collapses into three.

## Agent frontmatter

```yaml
---
name: tier2-routine
description: When Claude should pick this agent.
model: sonnet
effort: medium
tools: Read, Grep, Glob
---
```

- `model` accepts the aliases `haiku`, `sonnet`, `opus`, `fable`, and `inherit`, or a full
  model id such as `claude-opus-5`. Aliases survive model releases; full ids do not. The
  roles here use aliases.
- `effort` accepts `low`, `medium`, `high`, `xhigh`, `max`. Which levels are actually
  available depends on the model.
- `tools` allowlists; `disallowedTools` blocklists. Read-only roles here use
  `tools: Read, Grep, Glob`. Note that granting `Bash` to a "read-only" reviewer is a
  prompt-level restriction, not a permission-level one — `tier3-reviewer` additionally sets
  `disallowedTools: Write, Edit`.
- `maxTurns` caps the agent's turns. Useful on tier 1 so a mechanical job cannot loop.
- Plugin-shipped agents cannot set `hooks`, `mcpServers` or `permissionMode`.

## Parallel writes

When several subagents write to the same repository, `isolation: "worktree"` gives each one
its own git worktree, and the main agent merges afterwards. This is a safety net, not a
substitute for the non-overlapping write scopes required by section 3 of `SKILL.md` — two
agents editing the same file in two worktrees still produces a conflict you have to resolve
by hand.

## Where skills live

- Personal: `~/.claude/skills/<name>/SKILL.md`
- Project: `.claude/skills/<name>/SKILL.md`
- Plugin: `<plugin>/skills/<name>/SKILL.md`

A plugin skill is always namespaced: this one is `/dirty-work:dirty-work`. Plugin
agents appear in `/context` under Custom Agents and are @-mentioned by their scoped name.

## Keeping the main agent light

- Launch independent subagents in one message so they run concurrently.
- Background a subagent unless your very next action depends on its result.
- A subagent's final report is what reaches you. Ask for conclusions; anything you ask it
  to paste back lands in your context.

## Sources

- <https://code.claude.com/docs/en/plugins-reference>
- <https://code.claude.com/docs/en/plugin-marketplaces>
- <https://code.claude.com/docs/en/sub-agents>
