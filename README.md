# Change management — a Claude skill for manual changes

A [Claude](https://claude.com) skill that walks an operator through a **manual change**: one that can't go through a CI/CD pipeline, especially manual database work (running scripts against environments, parameter changes, version upgrades, failovers). It takes the change from plan, through review, to controlled execution and verification, leaving an auditable record in git.

It is built on cognitive-systems-engineering and safety-science principles: make the operator think before acting, keep them on track while acting, capture what *actually* happened (including the adaptations the plan didn't anticipate), and route any divergence back for review so the process learns from its own misses. The same ideas behind how aviation and other high-consequence fields run changes.

**It does not run the change.** The operator runs every command. The skill produces the record, guides the steps, and records what was observed. People make the decisions: peers and a Bar Raiser review, a change board approves escalated changes, the operator judges each step.

## Why

Most production changes should flow through a pipeline, with its tests, gradual rollout, and automatic rollback. Some can't: manual database operations, infrastructure migrations, one-off platform work. On those, there's no automated safety net, so the person making the change *is* the safety system. They're often working from memory or an out-of-date document, with no shared record of what was planned, what was actually done, or who agreed it was safe. This skill gives that work the same discipline a pipeline would, without requiring one.

## What it does

1. **Write it up.** One question at a time, it walks the pre-flight (what's changing, customer impact, failure and rollback, timing, awareness), pushing back on shallow answers and capturing the operator's hedges and assumptions honestly.
2. **Get it reviewed.** Locally by default (peers plus a Bar Raiser); escalated to a central change board (CAB) only when high-risk or cross-team. Review happens in git; the skill helps notify, records decisions, and never gates.
3. **Execute it.** Readiness checks first (a go/no-go pre-flight that verifies the conditions are right to start). Then read-do: present each step with its command in a copy-ready block, the operator runs it and pastes back the *actual* output, the skill records it and compares to expected. Hold points pause for a second person. Adaptations are captured, not fought. If it becomes a live incident, the skill says so and hands off to incident response.
4. **Verify it.** The mirror of the readiness checks: the system is back to normal *and* the customer can use the service again.
5. **Close the loop.** Went to plan → closed. Diverged → flagged for review-back; the record stays open until the reviewers' outcome (template fix, lesson, or postmortem) is recorded.

## The substrate

The skill works in a **git repository of change records**. Give it a repo (a local path or a remote URL) at the start, or it bootstraps a local one you can push to a remote later.

It uses only basic git (`init`, `add`, `commit`, `push`, `pull`). It does not open pull requests, run `gh`, or call any hosting provider's API. Two reasons: it works the same on any host (GitHub, GitLab, Bitbucket, a plain git server), and review lives on the record itself, not in a platform's pull-request UI, so the skill produces and commits the record and leaves anything platform-specific to you. A team on GitHub can of course open a pull request around a committed record if they want; that's their step, not the skill's.

The git history is the audit trail and the accountability record: named author, named reviewers, the executor, and what was observed at each step. The skill fills in what it already knows (the executor from the record header, timing from the commit) rather than asking the operator to log it by hand; it asks only for what carries real information.

## Requirements

A Claude client with [skills](https://docs.claude.com/en/docs/claude-code/skills) support (for example Claude Code), and `git` on the operator's machine.

## Install

Copy the `change-management/` folder into your skills directory:

```
cp -R change-management ~/.claude/skills/
```

Or per project: `<project>/.claude/skills/change-management/`. Then describe a change in plain language, for example "I need to make a manual database change for customer X", and the skill takes over.

## A worked example

[`examples/`](examples/) contains a filled-in change record (a Postgres connection-limit change) showing the bar for detail: each step says what to run, what to expect, and what to do if it's wrong. "Run the upgrade" with nothing else is not enough for someone else to execute or for a reviewer to challenge.

## Files

```
change-management/
  SKILL.md                        the workflow Claude follows
  template.md                     the change-record structure it fills
  reference/
    preflight-questions.md        the pre-flight questions, guidance, and shallow-answer red flags
examples/
  postgres-connection-limit.md    a filled-in record, for reference
```

## Status

Early (v0.1). Validated in dry-run testing (the happy path and a failed-execution / incident path). Not yet run against a real production change. Feedback and field reports are welcome; see [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

[Apache 2.0](LICENSE).
