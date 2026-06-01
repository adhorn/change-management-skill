# Contributing

Thanks for your interest. This is a small, focused skill, and the most valuable contributions are field reports and sharpened questions, not big features.

## What's most welcome

- **Field reports.** Did you run a real change through it? What worked, what got in the way, where did it ask the wrong thing or miss something? Open an issue describing the change (sanitised) and what happened.
- **Better pre-flight questions.** The pre-flight is the heart of the skill. If a question is unclear, missing, or lets a shallow answer through, propose a fix to `change-management/reference/preflight-questions.md`.
- **Clarity fixes.** Wording in `SKILL.md` that's ambiguous or that an LLM follows wrongly.

## Principles to keep

If you propose a change, keep it consistent with the skill's design:

- **The skill never runs the change.** The operator runs every command. The skill guides and records.
- **The skill never decides.** It surfaces, asks, and records. People approve, judge severity, and choose how to handle a mismatch.
- **One question at a time.** The operator is often mid-task and under pressure. Respect their attention.
- **Capture work-as-done, not just the plan.** Adaptations and the reasoning behind them are the most valuable thing the record holds.
- **Don't make the operator log what the tool already knows.** Ask only for what carries real information.
- **Plain git only.** No pull requests, no `gh`, no git-host API. Portability matters.

## How to propose a change

1. Open an issue first for anything beyond a typo, so we can agree the direction.
2. Keep pull requests small and focused on one thing.
3. Note in the PR what you changed and why.

## Licence

By contributing, you agree your contribution is licensed under the project's [Apache 2.0](LICENSE) licence.
