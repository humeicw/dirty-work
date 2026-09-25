# Trigger smoke test

Does Claude Code actually load the `dirty-work` skill when a request calls for it, and leave
it alone when it does not? This file records the two rounds of testing behind the numbers
on the front page: the method, the requests, the results and the limits.

It is a smoke test, not a benchmark. Read the [limitations](#limitations) before quoting any
number from it.

## Summary

Description tested: the one in `skills/dirty-work/SKILL.md` since 0.1.1 (it names the kinds
of work the skill covers instead of listing phrases). "Paragraph" means the standing
paragraph from [Making it trigger](../README.md#making-it-trigger) in the README.

| Round | Requests | Description alone | Description + paragraph |
|---|---|---|---|
| 1 (2026-09-19) | 6 should fire, 4 should not; **also used for tuning** | fired 3 of 12, false fires 0 of 8 | fired 12 of 12, false fires 0 of 8 |
| 2 (2026-09-25) | 6 should fire, 4 should not; **held out, never used for tuning** | fired 0 of 12, false fires 0 of 8 | fired 12 of 12, false fires 0 of 8 |

Each request was run twice per condition, so "of 12" means 6 requests times 2 runs, and
"of 8" means 4 requests times 2 runs.

What the two rounds show together: with the paragraph, the skill fired on every run of both
request sets, including six new requests it had never been tuned on, and never fired on a
request it should ignore. Without the paragraph, the description alone is not enough, and
on the held-out requests it did not fire once. That is why the README calls the paragraph
required.

## Method

### Setup

| Item | Value |
|---|---|
| Claude Code | 2.1.258 |
| Model | `--model sonnet`; every run's `init` event and usage report named `claude-sonnet-5` |
| Machine | Windows 11, one subscription account (no API key) |
| Plugin | loaded for the session only with `claude --plugin-dir <plugin directory>`; nothing installed |
| Test repository | a 14-file Python package (an order service: `orders`, `pricing`, `inventory`, `shipping`, `auth`, `config`, two test files, two docs, a `Makefile`, a `build.log` with one failed build) |
| Turns | `--max-turns 2` |
| Tools | read-only: `Read`, `Grep`, `Glob`, `Task`, `Skill`, `TodoWrite` and read-only shell commands allowed; `Write`, `Edit`, `NotebookEdit` and destructive shell commands disallowed, so the same repository could be reused unchanged |
| Runs | 2 per request per condition |

### Isolation

The test machine has other skills, subagents, a user-level `CLAUDE.md` and auto memory. Any
of them could make the model behave differently, so each run excluded them:

```sh
export CLAUDE_CODE_DISABLE_CLAUDE_MDS=1     # no user-level CLAUDE.md
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=1    # no auto memory
claude --plugin-dir "<plugin directory>" --setting-sources project ...
                                            # no user-level skills, subagents or settings
```

The two environment variables are not documented. They were found in the Claude Code
binary and may change between versions, so each round started with a probe that must
answer `NONE`:

```sh
claude --plugin-dir "<plugin directory>" --setting-sources project --model sonnet \
  -p "Answer from your loaded context only; do not use any tool and do not read any file. Is there a user-level global CLAUDE.md or personal memory in your system context right now? If yes, quote its very first markdown heading line verbatim. If no such file is in your context, reply with exactly: NONE" \
  --output-format stream-json --verbose --max-turns 1
```

With the isolation in place, the probe answered `NONE`, the `init` event showed no memory
paths, and the only skills loaded were the built-in ones plus `dirty-work:dirty-work`.
Without the environment variables, the same probe quoted the first heading of the
user-level `CLAUDE.md`.

`--safe-mode` cannot be used for this test: with `--plugin-dir` it keeps the plugin listed
but drops the plugin's skill and subagents.

### The two conditions

1. **Description alone**: the plugin as shipped, nothing else.
2. **Description + paragraph**: the same, plus the standing paragraph from the README,
   passed with `--append-system-prompt "<paragraph>"`. A separate probe confirmed that the
   model could quote the paragraph's first sentence verbatim from its context.

### One run

```sh
claude --plugin-dir "<plugin directory>" --setting-sources project --model sonnet \
  [--append-system-prompt "<paragraph>"] \
  --max-turns 2 --output-format stream-json --verbose \
  --allowedTools Read Grep Glob Task Skill TodoWrite \
    "Bash(ls:*)" "Bash(git status:*)" "Bash(git log:*)" "Bash(git diff:*)" \
    "Bash(grep:*)" "Bash(rg:*)" "Bash(find:*)" "Bash(cat:*)" "Bash(head:*)" \
    "Bash(tail:*)" "Bash(wc:*)" \
  --disallowedTools Write Edit NotebookEdit \
    "Bash(rm:*)" "Bash(mv:*)" "Bash(sed:*)" "Bash(python:*)" \
  -p "<request>" < /dev/null > out.jsonl 2>&1
```

### Scoring

A run counts as **fired** when the stream-JSON output contains an `assistant` message from
the main thread (no `parent_tool_use_id`) with a `tool_use` block whose `name` is `Skill`
and whose `input` mentions `dirty-work`. The model reading `SKILL.md` with `Read` does not
count, and nothing the model says about itself counts.

Both rounds were checked a second way, by a plain text search of the raw output files for
`"name":"Skill"` together with `dirty-work`. Both methods gave the same set of fired runs.

Every fired run in both rounds fired on the first turn, before the model read any file. In every run the skill was registered in the `init` event, so no miss was a
loading failure.

## Round 1 (2026-09-19): the tuning set

These requests were written first and then used over several rounds to tune the
description and to draft the standing paragraph. Their numbers are therefore optimistic.

### Requests

Should fire:

| ID | Kind of work | Request |
|---|---|---|
| T1 | investigation | `the build broke after last week's merge and I can't work out why. dig into it and tell me what happened.` |
| T2 | code search that sounds small | `quick one: does anything other than orders.py use the inventory module?` |
| T3 | logs and bulk reading | `go through the build logs and the docs folder and pull out everything that's relevant to the pricing failure.` |
| T4 | bulk reading that sounds small | `just have a quick skim of the repo and tell me whether it's in decent shape.` |
| T5 | one-off rewrite | `rename reserve to hold_stock everywhere in the project, including the tests and the docs.` |
| T6 | build across several files | `add a discount-code feature: a new module for the codes, wire it into create_order, and add tests for it.` |

Should not fire:

| ID | Kind | Request |
|---|---|---|
| N1 | one named edit (typo) | `there's a typo in src/pricing.py - the comment says "dicount", it should be "discount". fix it.` |
| N2 | explain one named function | `what does the quote() function in src/shipping.py do?` |
| N3 | general question | `in python, what's the actual difference between a tuple and a list? just curious.` |
| N4 | document co-written turn by turn | `let's write the release notes for 0.3.0 together. draft a one-paragraph summary first and I'll tell you what to change.` |

### Results for the current description

| Request | Description alone | Description + paragraph |
|---|---|---|
| T1 | no / yes | yes / yes |
| T2 | no / no | yes / yes |
| T3 | no / no | yes / yes |
| T4 | no / no | yes / yes |
| T5 | yes / yes | yes / yes |
| T6 | no / no | yes / yes |
| N1–N4 | no on all 8 runs | no on all 8 runs |
| **Fired** | **3 of 12** | **12 of 12** |
| **False fires** | **0 of 8** | **0 of 8** |

### Earlier descriptions, same requests

| Description | Alone | With paragraph | False fires |
|---|---|---|---|
| before any rewrite | 0 of 12 | not tested | 0 of 8 |
| phrase list (shipped in 0.1.0) | 8 of 12 | 12 of 12 | 0 of 8 |
| kinds of work (0.1.1, current) | 3 of 12 | 12 of 12 | 0 of 8 |

The phrase-list description did better on its own because it quoted phrases close to these
very requests. It was replaced in 0.1.1 because no list of phrases is ever complete; with
the paragraph, both reach 12 of 12. The "alone" figure for the phrase list was measured on
a draft that differs from the shipped text by three punctuation and wording changes; the
shipped text was spot-checked on the four requests the draft had caught, and fired on all
four.

Three extra runs placed the paragraph in a real `CLAUDE.md` in the test repository instead
of injecting it (T2, T6, N4, with the phrase-list description). All three matched the
injected result. Those three runs also loaded the user-level `CLAUDE.md`, because the
environment variable that removes it removes every `CLAUDE.md`, so they are a spot check,
not a clean comparison.

## Round 2 (2026-09-25): held-out requests

Ten new requests, written after tuning had finished and never used to change the
description or the paragraph. They cover the same four kinds of work the description
names, with a different request in each slot, and the four should-not-fire requests were
chosen to sit close to the line.

The isolation probe was re-run first and answered `NONE`, with the same skill and subagent
lists as in round 1.

### Requests

Should fire:

| ID | Kind of work | Request |
|---|---|---|
| H1 | investigation: why something happens | `a customer says their order came through with a total of 0. work out how that can happen in this code.` |
| H2 | investigation: what a change would affect | `I want price_for to return whole cents as an int instead of a float. before I change anything, tell me everything that would be affected.` |
| H3 | state of the codebase, sounds small | `real quick - which parts of the code have no tests at all?` |
| H4 | one-off rewrite across many places | `pull every hard-coded number out of the modules (prices, rates, stock levels) into config.py and make the modules read them from there.` |
| H5 | migration | `we're dropping the Makefile. move the build and test setup over to a pyproject.toml and update anything that still mentions make.` |
| H6 | build across several files | `add persistent sessions: keep logged-in sessions in a sqlite file instead of the in-memory dict, keep login() and check() working the same, and add tests.` |

Should not fire, each close to the line:

| ID | Excluded because | Request | Why it is close to the line |
|---|---|---|---|
| M1 | one edit at a place the user named | `in src/shipping.py, make quote() raise a ValueError that names the unknown method, instead of letting the KeyError through.` | a real behaviour change, not a typo |
| M2 | answerable from what is in front of the model | `what does this log line mean? "[10:02:15] ERROR src/pricing.py: AssertionError in test_pricing"` | the line comes from this repository's failing build, the topic of several should-fire requests |
| M3 | document co-written turn by turn | `I'm writing a short CONTRIBUTING section with you, one paragraph at a time. here's my first go: "Run make test before you open a PR, and keep changes small." tighten it up and then I'll send the next bit.` | it mentions the build setup |
| M4 | general question | `why do people say you shouldn't use floats for money?` | the test repository computes prices with floats |

### Results

| Request | Description alone | Description + paragraph |
|---|---|---|
| H1 | no / no | yes / yes |
| H2 | no / no | yes / yes |
| H3 | no / no | yes / yes |
| H4 | no / no | yes / yes |
| H5 | no / no | yes / yes |
| H6 | no / no | yes / yes |
| M1 | no / no | no / no |
| M2 | no / no | no / no |
| M3 | no / no | no / no |
| M4 | no / no | no / no |
| **Fired** | **0 of 12** | **12 of 12** |
| **False fires** | **0 of 8** | **0 of 8** |

With the description alone, the model's first step on the should-fire requests was to start
working itself: `Bash` or `Grep` in 11 of 12 runs. In the remaining run (H1, second run)
it handed the investigation to the built-in `Explore` subagent straight away, without
loading the skill; by the scoring rule that is not a fire. With the paragraph, the first step was the
`Skill` call in all 12 runs. None of the 40 runs ended in an API error.

## Limitations

- **Small sample.** Two runs per request, 20 runs per condition per round. Twelve of twelve
  has a 95% lower confidence bound of about 74%; zero of twelve has a 95% upper bound of
  about 26%. Read 12 of 12 as "usually", not "always".
- **One model.** Sonnet only. Opus, Haiku and other tools (Codex and others) are untested.
- **Single run.** Each condition was run once per round; no repeat on another day, machine
  or account.
- **One small repository.** A 14-file Python package. Larger repositories might change the
  model's judgement in either direction; that is untested.
- **Two turns.** `--max-turns 2` only shows whether the skill is loaded at the start. Whether
  the model would load it later in a longer session is not measured.
- **Injected paragraph.** In both rounds the paragraph was passed with
  `--append-system-prompt`, which places it differently from a real `CLAUDE.md` or
  `AGENTS.md`. Only three runs in round 1 used a real `CLAUDE.md`, and those were not
  clean (see above).
- **Round 1 was tuned on its own requests.** The descriptions and the paragraph were revised
  while looking at round 1 failures, so round 1 numbers are optimistic. Round 2 exists to
  check them.
- **Round 2 requests were written for this project**, by an agent that had read the
  description and the round 1 requests, not by independent users. They are held out from
  tuning, not independent. There are only four should-not-fire requests.
- **Undocumented isolation.** The two environment variables are not a documented interface.
- **Untested combination.** The paragraph with the pre-rewrite description, or with no
  description change at all, has not been measured.

If the skill failed to fire for you on a request it should cover, or fired when it should
not, please open an issue with the exact request you typed. Real requests are the data this
test is missing.
