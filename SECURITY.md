# Security

This repository ships Markdown only: the skill, two reference documents, seven subagent
definitions and two JSON manifests. There is no executable code, no install step and no
network call. Installing copies text files onto your machine and nothing runs. You can read
everything that reaches you in a few minutes, and you should.

Text is not automatically harmless, though. This skill makes your agent send more work to
subagents, and a reader subagent receives whatever you point it at. Its own rules say a
work order may never widen a subagent's authority, but that is prose, not a sandbox. Your
permission settings are the enforcement.

**Reporting.** If you find something that could harm users of this repository — a
prompt-injection path, an instruction that would make an agent leak data or widen its own
permissions, or a tampered file — do not open a public issue. Use GitHub's **private
vulnerability reporting** on this repository: the *Security* tab, then *Report a
vulnerability*.

I will reply as soon as I can. Valid reports get a fix, a new version, and a `CHANGELOG.md`
entry describing the problem. Only the latest version is supported. There is no bounty.
