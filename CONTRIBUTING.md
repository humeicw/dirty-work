# Contributing

This repository is one skill, written in Markdown. No scripts, no build step. Contributing
is writing and testing, not coding.

## Most useful

1. **"It did not trigger."** The skill was installed, the situation clearly called for it,
   and the agent never loaded it. Include the exact prompt. This is one of the most common
   failure modes for skills, and I cannot fix what I cannot reproduce.
2. **"It triggered and then did the wrong thing."** Say what the agent decided and why it
   was wrong.
3. **Install instructions that are wrong for your tool.** Say which tool and version.
4. **Wording a non-native English reader has to read twice.** Shorter and plainer is a real
   improvement here, not a nitpick.

## Not accepted

- Scripts, hooks, a CLI or a daemon. Staying text-only is the point.
- A second skill in this repository. One repository, one skill.
- Hard-coded model names in the tier table. That column is for each user to fill in.
- Claims without evidence: numbers, benchmarks, or "tested on X". Where the README says
  something is untested, that is deliberate.

## If you change what the skill does

- Bump `version` in **both** `.claude-plugin/plugin.json` and
  `.claude-plugin/marketplace.json`. Users are only offered an update when it changes.
- Add an entry to `CHANGELOG.md`.
- Say how you tested it: which tool, which model, what you observed.
- Changing the `description` field changes when the skill triggers. Treat it as a behavior
  change and say what should and should not trigger it afterwards.
- `skills/dirty-work/SKILL.md` stays under 500 lines and roughly 5,000 tokens; the whole
  body loads every time the skill fires. Tool-specific mechanics go in `references/`.
- Keep `README.md` and `README_ZH.md` in step. If you only read one of the two languages,
  change that one and say so.
- Run `claude plugin validate ./ --strict` before opening the pull request.

Contributions are licensed under the MIT License, same as the rest of the repository.
