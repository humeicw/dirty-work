---
name: dirty-work
description: Read this before the first file read, search or shell command of a task. It decides what goes to a subagent instead of you, how to write the work order, how to verify what comes back, and which model tier each job needs. It covers four kinds of work at any size - investigation (finding out why something happens, where something is used, or what state a codebase, a set of files or a pile of logs is in), one-off rewrites that touch many places (renames, refactors, format changes), migrations, and self-contained builds that span several files. A task that sounds tiny still counts when answering it means searching or reading beyond what is already in front of you. It does not cover one edit at a place the user has named, a question you can answer from the current context, or a document you and the user are writing together turn by turn. The user can override it at any time.
---

# Dirty Work

Let someone else do the dirty work. The digging, the grepping and the bulk reading go to a
subagent. The responsibility does not: you still write the work order, judge what comes
back, and answer for the result.

Your context window is the resource you run out of first. Reading a repository yourself
costs the rest of the session; asking a subagent to read it and report back costs a
paragraph.

Section 2 says which work must be delegated. If the user tells you to do it yourself, or
names a model, follow the user instead.

## 1. Priority order

Fixed: **get the task right > protect the main agent's context > save tokens, quota and
time.**

Delegate whole tasks. Pick a tier by how much judgment the job needs. Split a task only
when the parts are genuinely independent and parallelism buys something real — every split
costs another round of explaining the background. There is no fixed batch size.

## 2. What to delegate

The main agent coordinates: understand the user's goal, handle the project setup, the
conversation with the user and the permission calls, define who does what and what "done"
means, merge the results, and write the final answer. Coordinating is not the same as
executing.

A document you are writing with the user turn by turn stays with you. Anything inside it
that can be investigated, coded or built independently still goes out.

Everything else that can be executed independently — investigation, search, analysis,
coding, testing, implementation — goes to a subagent by default, with the full
responsibility for that piece.

Two rules that do most of the work:

- **Investigation goes out.** Anything where you would have to hunt (which file is
  authoritative, who owns this, what did we miss) is investigation. Hand it over with the
  open questions attached. Do not investigate in order to write the work order.
- **Bulk reading goes out.** When you need to know what is in a large body of material —
  source documents, logs, transcripts, a whole data file — send a reader subagent (usually
  tier 2) and ask for only what this task needs. You read the project entry point and the
  state file. You do not read the pile.

Before writing the work order, settle only four things: the goal, the authority, the scope,
and what you already know or suspect about where to look. If that is enough to describe the
job, send it. Unknowns are the subagent's job to resolve inside the task; never promote an
unknown to a fact to make a work order look complete.

## 3. The work order

A work order is a task description for the executing agent. No fixed form, no separate
file. Give it the goal and a map of where to look — not a replay of your own thinking.
Except for external contracts, fixed interfaces, safety boundaries and verified risks, the
subagent decides how to do the work.

Include what is relevant:

- **Goal and done.** Deliverable, acceptance criteria, scope of responsibility, stop
  conditions, and the specific answer or artifact you need back.
- **Map, not conclusions.** Where the material is, what to search, locating hints, and any
  current decision, constraint, dependency, fact boundary or recent change that would
  change the subagent's choices. Open questions can go straight in. The subagent resolves
  them itself; only when the given scope still cannot produce an input, authorization or
  constraint it needs to continue does it stop and reply `NEEDS_CONTEXT` with the smallest
  missing list.
- **Authority and write scope.** What is authorized, which boundaries it must not cross,
  and which files, modules or areas it exclusively owns for a writing task. **A work order
  never widens authority.** Cost, accounts, publishing, sensitive data, production systems,
  and deleting or overwriting the user's files still require the user's confirmation under
  whatever rules your setup already has. Parallel write scopes must not overlap; when
  there is shared state or an ordering dependency, serialize. Preserve other people's
  changes.
- **Required reading and reporting.** Any document or skill that would change how the job
  is executed, judged or authorized, plus the checks needed to finish and the shape of the
  reply. An information task reports the range it actually covered. When the result depends
  on real hardware, a live service or a production state, checking it directly is part of
  "done" — unchecked means the gate stays failed.

**This skill applies at every level.** A subagent that needs to delegate applies the same
rules: same judgment, same tiering, same work order, same verification. It does not need
separate permission in its own work order. Every work order should point at this skill, and
the receiving agent reads it before deciding whether to delegate. Throughout this file,
"main agent" means whoever is delegating and "subagent" means whoever receives. Every level
is still bound by the user's explicit limits, the authority in the original task, and the
limits of the runtime.

Pass down the minimum history the task needs. Ask for back only what you need to move
forward. Long process narratives and raw material stay in the subagent's thread. Write files
only when that is authorized and the file is genuinely worth reusing.

A long-lived subagent should hold only reusable context — rules, criteria, background.
One-off bulk material does not belong in it: use thumbnails for images, spot-check writes
with a script, and for material that truly has to be read item by item, hand batches to
short-lived grandchild agents that end once they report. For hundreds of similar item-level
judgments, measure the per-item cost on a small batch first, then decide the batch size and
whether to continue. Write results per batch; do not finish everything and then read the
whole file back.

## 4. After you delegate

- **Follow up when you need to, not out of anxiety.** While a subagent runs, wait for its
  report. Query it only when the user asks, when something looks wrong, or when your next
  decision needs its progress. Ask for the one state or difference you need — do not re-read progress
  files, logs or intermediate artifacts just because you are waiting. If something needs
  digging into, give it back to the agent that owns it and take back conclusions, the
  evidence that matters and open decisions. Wait with a single long timeout; when it
  returns, do not read files or list agents, just keep waiting until the report arrives or
  something actually looks wrong. Stay light while waiting, and do not preload material
  before delegating.
- **No duplicate work, no double-assigning.** While a subagent runs, do not perform the
  same search, read, analysis or implementation, and do not give substantially overlapping
  work to another agent. Non-overlapping integration work is fine.
- **Take information reports at face value.** Do not re-audit the content of an information
  or analysis task, and do not go read the source material again. Your conclusion covers
  only what the report actually examined. If a report materially contradicts something you
  know, or contradicts another report, in a way that changes a decision, send the conflict
  to an independent agent — at least as capable as the original, higher tier when the
  conflict is complex or risky. Unknowns and `NEEDS_CONTEXT` are the current boundary. A
  new question is a new assignment, not a reason to re-audit the old answer.
- **One owner at a time.** You may take back ownership of an implementation only after the
  subagent completes, fails, blocks, or is explicitly stopped — and say why. Never two
  owners for the same piece of work.
- Check only the subagents that matter right now. A truncated tool result means the return
  was incomplete, nothing more: paginate, read a targeted range, raise the output limit,
  switch to a compact format, or delegate — until you have enough.
- **An investigation subagent is a cache you can question again.** Reporting ends the call,
  not the agent. Keep its address, what it read and what it covered in a lightweight task
  map. A follow-up question on the same subject goes back to the same agent. If it can no
  longer be reached, delegate afresh for the current task — do not build machinery to
  restore the cache.

## 5. Parallelism and resources

- Safe to parallelize: independent read-only exploration, tests, extraction and diagnosis.
  Parallel writing is safe only once section 3 has given each agent a non-overlapping,
  exclusive scope.
- The main agent merges conclusions across agents and resolves conflicts. When the channel
  or runtime state is unknown and a bulk failure would be expensive, send one small probe
  first.
- When a single round of delegation would clearly exceed what the user expects to spend or
  what they asked for, say how many agents, which models, and why — before you send them.

## 6. Verification and reviewers

- The executing agent runs the tests and checks its task agreed on, and reports what it
  actually covered. Any gate it did not complete stays failed.
- Read-only information tasks are accepted as they are (section 4). When you need
  independent confirmation of a write, an implementation, a file or an external state,
  prefer mechanical verification — tests, schema checks, lint, exit codes, querying the
  target state — and report compact results. When the mechanical signal is not enough and
  the risk is worth it, send a separate reviewer. Do not re-read the implementation
  yourself to redo the acceptance check.
- Use a reviewer when the user asks for independent opinions. A reviewer finding the work
  unacceptable is a successful review, not a failed one.

## 7. When results fall short

Missing an angle is a reason to add an agent with a new perspective.

When the same subtask comes back wrong twice in a row at its current tier, or still misses
the agreed acceptance criteria after a fix, **escalate to a higher tier** and hand over the
failure evidence and what was already tried. Escalate earlier if you already have clear
evidence the tier is not capable enough.

Count task reports, not incidents: a single tool error during execution does not count, and
a reviewer finding problems in the thing it reviewed does not count. Channel, quota,
permission and environment failures do not count toward capability escalation either —
clear the block, switch to an equally capable channel, or hand it back to the main agent to
reassign.

At the top available tier, or when the user has pinned a model, hand it back to the main
agent to change the plan or explain the blocker. Do not hide a design or environment
problem with endless escalation, endless reviewers or endless grandchild agents.

## 8. Choosing a tier

Tier selection asks one question: **how much of this job is not written down in the work
order and has to be decided by the subagent?**

Check the tiers in this order — **4, then 3, then 2, then 1 — and stop at the first hit.**
Number of files, length, how important the project is, and the category name of the task
("research", "coding", "review") do not decide the tier by themselves. Editing an important
config file to a given diff is not the same as deciding what that config should say. If the
user names a model or asks for maximum capability, follow the user; if the model turns out
not to be capable enough mid-task, use section 7.

| Tier | When it applies | Typical jobs | Your model (fill this in) | Example: Claude Code |
|---|---|---|---|---|
| **1 — Mechanical** | The work order already states the input, the processing rule and the output. The subagent does not need to understand what the content means; the result can be checked directly. | Replace terms from a list; inventory against a checklist; convert formats; run a given command and report verbatim; apply a given diff. | cheapest / fastest model | `haiku` |
| **2 — Understanding, local judgment** | Goal and scope are fixed, but the content has to be understood before each item can be judged. The judgments are independent — getting one wrong does not break the others — and no overall design has to be held in mind. | Read material and report what is relevant; work through a checklist; classify items; edit within named files following a stated approach; write and pass tests as described; a minimal fix when root cause and remedy are already known. | mid model, medium reasoning | `sonnet`, effort `medium` |
| **3 — Multi-step reasoning inside a fixed direction** | The approach (or the question) and the acceptance criteria are already set, but the steps interlock: each step depends on the last, and one consistent design or line of reasoning has to be maintained throughout. | Implement an agreed design across modules; reason from located evidence to a root cause or an acceptance verdict; produce a design within fixed constraints; find counterexamples and risks in an implementation. | strong model, high reasoning | `opus`, effort `high` |
| **4 — Exploration and pivotal decisions** | Any one of: the work order can only state a goal because the path has to be discovered as you go, with multiple tool rounds, hypotheses and plan changes; the agent has to decide project goals, shared rules, system responsibilities or permission boundaries; a mistake is hard to undo and could mean lost data, exceeded authority or a production outage. | Debugging a failure with no known source; designing something with no precedent; setting shared rules; decisions touching production or accounts. | the strongest model you have, highest reasoning | `fable`, effort `xhigh` — or your strongest model at its highest effort |

**Fill in the model column once, for your own lineup.** The tier definitions are the part
that transfers; model names change every few months. Pick one model per tier from whatever
you actually have access to, cheapest first, and write them into your own copy. The Claude
Code column is an example, not a recommendation.

**Tier 3 and tier 4 must not end up identical.** If you only have one top model, separate
them by reasoning effort; otherwise the table is really three tiers with four names.

**Re-tier as you go.** A tier does not carry through a whole task just because the opening
move needed it. Once tier 4 has settled the path and the approach, and once tier 3 has
settled the plan, the rules and the acceptance criteria, judge the remaining work again:
agreed implementation to tier 3, scoped local edits to tier 2, replacement, inventory and
running commands to tier 1.

Reading material and extracting the relevant points is tier 2, no matter how long it is.
Tier 1 handles bulk material with a script or a command instead of reading it into the
model. When a job genuinely requires the model to read it all — applying a given diff by
hand to a large file, comparing two lists line by line — and the material is over roughly
200 KB (about 50,000–80,000 tokens), move it up to tier 2.

### Roles do not replace tiering

Each role template is pinned to one tier. The name only describes its specialty and its
permissions:

- `auditor` — tier 2, read-only. Checks item by item against fixed rules, a checklist and
  run evidence. Reports; never fixes.
- `iteration` — tier 2, repair. Use only when you already have concrete failure evidence, a
  root cause and a repair direction, and the task authorizes the fix.
- `reviewer` — tier 3. Hunts for risks, counterexamples and omissions in an implementation
  or a plan. It may run tests and read-only commands; it never modifies what it reviews.

When checking, fixing or reviewing needs higher-tier judgment, use the executor role for
that tier instead and write the read-only or repair boundary into the work order. The main
agent's coordinating duties still follow section 2.

Role files define specialty, model, reasoning effort and permissions. Delegation judgment,
work orders, ownership and acceptance follow this file. If you need a model or effort
different from a template, do not use the template — specify the model and effort
explicitly and keep the same authority boundaries in the work order.

The dirty-work repository ships seven ready-made Claude Code roles — `tier1-mechanical`,
`tier2-routine`, `tier2-auditor`, `tier2-iteration`, `tier3-complex`, `tier3-reviewer`,
`tier4-frontier`. `references/claude-code.md` says what each one is for and how to install
them if you only have the skill folder.

## Tool-specific mechanics

Everything above is tool-independent. The mechanics — where role files live, how to isolate
parallel writes, how to check and control whether a subagent inherits the parent
conversation and model — differ per tool:

- Claude Code: `references/claude-code.md`
- Codex: `references/codex.md`

If your tool is not listed, the rules above still apply; only the mechanics change.
