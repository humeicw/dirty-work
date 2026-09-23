# How these rules got this way

Dirty Work is the public version of a private skill I have used daily since July 2026.
Each rule is there because something went wrong first.

I list only changes I can back with a file: a commit, a same-day decision note, a session
log or a post-mortem. Results are real numbers from my own session logs where I have
them. Where I have none, it says so. The logs are observational: one person, one machine,
no controlled comparison.

My note from day two (2026-07-10) still heads §1: saving tokens is one thing, a clean
main context ranks higher.

§ numbers refer to [`SKILL.md`](skills/dirty-work/SKILL.md). [中文版](HISTORY_ZH.md)

## 1. 2026-07-10 to 07-14 — Twenty subagents, all on the most expensive model

- **The problem.** The tier table covered only one way of spawning subagents. On 07-10 an
  audit run through workflow orchestration launched 20+ subagents. None named a model, so
  every one inherited the main session's, my most expensive. I added a rule the next day.
  On 07-14 it happened again with 12 agents: the rule sat in a document nothing had read.
- **What I changed.** Every dispatch names its model. Shipped role files pin theirs. §8:
  name model and effort explicitly outside the templates. `references/codex.md`: check
  what a subagent inherits. Privately I also ran a hook that blocked model-less
  dispatches, until 09-02.
- **Result.** Then: every subagent in both incidents ran on the top model. Now: in my
  logs for 09-05 to 09-19, 2 of 123 dispatches (1.6%) did.

## 2. 2026-08-03 to 08-09 — A failure is not proof you need a smarter model

- **The problem.** A failed check meant "up one tier", unconditionally. Replies were
  capped at 10 lines. A reviewer's rejection was logged as a failed delegation. On 08-07
  two routine tasks were escalated to the top model. One tier showed a 44% pass rate;
  three of its "failures" were reviewers correctly rejecting bad work. The cap squeezed
  out information the main agent needed.
- **What I changed.** §7: count task reports, not incidents; environment failures don't
  count. §6: a reviewer's rejection is a successful review. §3: ask for what you need, no
  fixed length.
- **Result.** Top-model dispatches are now rare (2 of 123, see 1). Replies stayed short
  without a cap: the median report in those logs is 272 tokens.

## 3. 2026-08-11 to 09-02 — The control plane outgrew the work

- **The problem.** About 100 lines of rules, a work-order file and log line per dispatch,
  three hooks, a repair ticket per failure, and fresh-agent re-verification. In one
  project, small infrastructure gaps (a path, JSON, stdin) each set off the whole chain:
  ticket, fix, re-acceptance, verification, another review. No stop line. My 08-12
  write-up says time and tokens "far exceeded the value of the task". I have no total.
- **What I changed.** I rewrote the rules as 33 lines for one tool, and on 09-02 retired
  the machinery everywhere. What survives: §3 "No fixed form, no separate file"; §7 "Do
  not hide a design or environment problem with endless escalation". No scripts ship.
- **Result.** A dispatch is now one call. Seven work orders measured on 09-03 averaged
  about 365 tokens each. Before, each one also meant a file, a log entry and a hook
  check. The ticket loops ended with the tickets.

## 4. 2026-09-02 — "The task looked small"

- **The problem.** Whether to delegate was the main agent's call. An agent read the
  rules, judged a bug diagnosis small and did it itself. It became log tracing,
  cross-module diagnosis, code changes and tests. One turn used about 200K of a 258K
  window. The next opened with a compaction, and the fresh findings were lost. The agent
  said it had broken no hard rule. True: the rule was too soft.
- **What I changed.** §2: independently executable work goes out by default. "It looks
  small" and "I've already started" are not reasons. §4: an investigation subagent is a
  cache you can question again.
- **Result.** Not enough. The next day showed why.

## 5. 2026-09-03 — Four hours, zero delegations

- **The problem.** My always-loaded rules said: read the delegation skill "when you first
  consider delegating". A large clean-up task then ran about four hours with zero
  delegations. Output from 61 shell commands landed in the main conversation, which grew
  from about 65K to about 290K tokens over 89 requests. The agent's explanation: it never
  "considered" delegating, so it never read the skill. (Re-checked against the log on
  2026-09-19.)
- **What I changed.** A rule that isn't loaded can't fire, so the trigger moved into text
  the agent always sees: the `description`'s first sentence, plus the standing paragraph
  the README has you add. That day I also wrote the script behind the later reviews.
- **Result.** Smoke test on the public version: description alone fired 3 of 12; with
  the paragraph, 12 of 12; false fires, 0 of 8. Small, self-tuned sample. Over the next
  14 days, 21 of 35 sessions delegated. In those, 88.2% of file reads and shell commands
  ran inside subagents; on 09-03 it was none. In the 19 with complete logs, subagents
  handled about 19.0 million tokens and sent back about 41,000 (0.21%). I haven't checked
  whether the other 14 sessions should have delegated.

## 6. 2026-09-03 to 09-06 — Preparing the work order became the investigation

- **The problem.** The rules allowed a "minimal read-only probe" before writing a work
  order. A review found one diagnosis session's delegation in order, yet the main thread
  still pulled in too much one-off history, and wrote an unverified inference into a work
  order as fact. A subagent disproved it. Verdict: the agent took "prepare the work order"
  to mean "understand the background first".
- **What I changed.** A work order needs goal, authority, scope and leads, not the
  answer. §2: "Investigation goes out"; "Do not investigate in order to write the work
  order"; never promote an unknown to a fact.
- **Result.** One example, not a measurement: in a bug-fixing session on 09-06 the main
  agent read 3 files itself, while 24 subagents made about 2,800 tool calls over 4.5
  million tokens and sent back about 6,500. The behavior check the review asked for is
  still not run.

## 7. 2026-09-15 to 09-18 — 1.2 billion tokens in two days

- **The problem.** Nothing covered waiting, and a tier was chosen once, at the start. A
  two-day job sorting teaching material used about 1.2 billion tokens. Post-mortem: 63.7%
  went to 31,547 item-by-item verdicts at about 24,200 tokens each, all on the strong
  model. 20.8% went to coordination that produced nothing: of 703 main-window turns, 232
  waited, 119 re-read a progress file, 6 listed agents. OCR of 17,373 pages, by script,
  cost 1.6%. A 09-05 review had flagged polling; I had decided no rule was needed.
- **What I changed.** Parallelism wasn't the waste; per-item price and idle polling were.
  §4 "Follow up when you need to, not out of anxiety". §3: measure per-item cost on a
  small batch; long-lived subagents hold only reusable context; the skill applies at
  every level. §8 "Re-tier as you go".
- **Result.** No before-and-after yet. One example: a batch-sorting session on 09-16, the
  day of the post-mortem, made 10 dispatches, 7 to the tier-1 role and 3 to the tier-2
  role; 1.4 million tokens handled, about 2,700 sent back. And one miss: a review that week
  found an agent reused for three rounds until its cumulative input hit 48.8M tokens.
  Rule present, not followed.

## 8. 2026-09-06 to 09-18 — Tiers: from a price list to one question

- **The problem.** The first table (07-09) had six tiers ordered by model price. It went
  to seven, then three, then four on 09-06 with a fixed checking order. One strong model still covered
  tiers 2 and 3, and it was overused: in my logs for 09-05 to 09-19, 96 of 123 dispatches
  (78.0%) ran on the tier-3 model. The two cheap tiers got 20.3%.
- **What I changed.** A tier isn't a task type, a file count or a price. §8: one
  question (how much must the subagent decide for itself?), one model per tier, roles
  pinned to tiers.
- **Result.** No after-number yet. The 78% is the baseline, from before and around the
  change.

## What is not here

Changes with no written reason I could find (such as when `NEEDS_CONTEXT` first appeared,
on 07-20) and incidents I could not tie to one rule. Left out rather than guessed at.
