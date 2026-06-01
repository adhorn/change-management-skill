---
name: change-management
description: >
  Guide a manual production or non-production change from plan, through review, to
  controlled execution and verification. Built for changes that can't go through a
  pipeline, especially manual database work (running scripts against environments,
  parameter changes, upgrades, failovers). Use when someone is preparing a change
  ("I need to make a change to X", "prep a database change for <customer>", "create a
  change record") OR is ready to carry out an already-approved change ("run the change
  for <customer>", "execute the approved change"). The skill produces the change
  record, commits it to a git repo of change records, helps get it reviewed (locally by
  default, escalated to a central review board (CAB) only when it's high-risk or crosses
  teams), then guides execution step by step, capturing what actually happened (including
  adaptations) and routing any divergence back for review. It does NOT make the change
  itself, and it does NOT replace local review, the CAB, or human judgment.
---

# Change management

Guide one manual change end to end: write it up, get it reviewed, execute it under control, verify it, close the loop. One change = one customer, one service, one environment.

The skill produces the artifact and runs the read-do execution. **People** make the decisions: peers review, the CAB approves, the operator judges each step. The skill never approves a change, never decides risk, and never runs a command itself.

## How to talk to the operator (read this first; it governs every phase)

The operator is often mid-task, under time pressure, sometimes in a maintenance window. Respect their attention. These rules are not optional; they are the difference between a tool people use and one they abandon.

- **One question at a time. Always.** Ask a single question, wait for the answer, then ask the next. Never present a list of questions, never number a batch, never ask "a few things." Even when the reference lists several questions in a section, ask them one by one.
- **Never bundle a correction (or confirmation) with the next question.** If you need to confirm a detail or fix something, do *only* that, and wait. The next question comes in its own turn, after the current thing is settled. Two things in one message forces the operator to hold both, which is the failure mode to avoid.
- **Don't lecture before asking.** A short framing sentence is fine; a paragraph of preamble is not. Get to the one question.
- **Reflect the answer back in one short line, then move on.** Confirm you captured it ("In-place upgrade, noted.") without re-explaining it.
- **Let the operator's words lead.** Use their terms. Only correct a detail when it would make the *record* wrong (e.g. the wrong product name), and when you do, that correction is its own turn, nothing else attached.
- **Accept a good-enough answer and keep moving.** Probe shallow answers where it matters (rollback, blast radius), but don't interrogate a clear answer to polish it.

If you catch yourself writing two question marks in one message, stop and split it.

## First: find or set up the repo, and pick the mode

Change records live in a **git repository of change records**. Establish it before anything else:

- **Ask the user for the repo.** A local path or a remote URL. If a remote URL isn't cloned yet, clone it. Ensure a `changes/` folder exists at the root (create it if missing).
- **If they have none:** create a `changes/` folder in the current directory and `git init` it. Tell the user it's local for now and can be pushed to a remote later (`git remote add origin <url> && git push`) to become the shared record; nothing in the workflow changes when they do.

Plain git only: `init`, `add`, `commit`, `push`, `pull`. **Never** open pull requests, call `gh`, or use any git-host API or web feature.

Then pick the mode from what the user is doing:

- **New change** → start at Phase 1.
- **Execute an approved change** → the change was written and approved earlier (often days ago, in a maintenance window now). Find the existing record under `changes/`, read its status, confirm it's `approved` (locally or by CAB), and start at Phase 3. Do not start fresh. If execution was already part-done (some steps have observed results), resume at the first incomplete step: re-read what's been recorded so far, don't repeat completed steps.

Read `reference/preflight-questions.md` before Phase 1, and keep `template.md` as the structure for the record.

## The record's lifecycle

Every record carries a `status` line. Most changes are approved by local review; only high-risk or cross-team changes escalate to the central CAB. Move the record through these states and commit at each transition:

```
draft → in-review → approved → executing → verified → closed
                                              └─ (if it diverged) flagged for review-back → closed
```

`approved` is reached one of two ways, recorded in the record: **locally approved** (the local director + lead-engineer review, the default) or **CAB-approved** (escalated changes only). The skill helps reach `in-review` and records the approval the user relays; it never approves. If the record is edited after it reaches `in-review` or `approved`, drop it back to `draft` and clear the approvals; the review no longer matches the content. "Flagged for review-back" cannot be set to closed by the operator alone; it needs a recorded review-back outcome (see Phase 5).

---

## Phase 1 — Write it up (produce the artifact)

The thinking phase. Goal: a complete, thought-through record before anything runs.

1. **Identity.** Collect title, customer, service, environment, author. Keep to one customer / one service / one environment. If the same change spans several, make one record each and note `cloned_from`.
2. **Pre-flight.** Walk the questions in `reference/preflight-questions.md`, section by section, with the guidance inline. **Probe shallow answers:** a rollback of "revert it" with no steps, a blast radius that names nothing, a window that suits the operator not the customer. Push for a real answer or an explicit "not applicable, because…". This is the point, not paperwork. As you go, **capture the reasoning, not just the answer.** When the operator hedges ("not sure, the account manager would know"), admits a gap ("no idea, never done it"), or states an assumption ("shouldn't need a snapshot"), record it in the **Reasoning & open assumptions** section of the record, with who could confirm it. These uncertainties are the most valuable thing the write-up produces: the reviewer and CAB weigh them, and the learning loop reads them if the change later diverges. Never silently upgrade a hedge into a confident answer; "assumed, unconfirmed" is the honest record.
3. **Execution checklist, in two parts.**
   - **Readiness checks (pre-flight / go–no-go)** first. Before any change step, build the checks that confirm the conditions are right to start, like a pilot's pre-flight checklist. These verify, they don't change: e.g. backup/snapshot confirmed present, system healthy before you touch it, maintenance mode ready, second person / on-call available, any open assumptions from the write-up resolved (the things flagged for the account manager, etc.). Each has a way to verify and a go/no-go expected result. If any is no-go, the change does not start.
   - **Change steps** next. The ordered steps the operator will run. Each: description, the exact command, expected outcome, rollback action, and whether it's a **hold point** (a second person confirms before continuing). One action per step. Mark every step's origin as `planned` at this stage.
4. **Verification checklist (mirror of the readiness checks).** Build the checks that confirm the conditions are back to normal and healthy, and the customer can use the service again, not just that the change ran. Two sides: system back to normal (re-run the readiness checks that mattered: health good, version as intended, maintenance lifted, nothing left degraded) and customer-side (at least one check confirms the customer can do what they could before, from their entry point).
5. **Write the record** to `changes/<customer>/<service>/<YYYY-MM-DD>-<slug>.md` from `template.md`. Use the date the user gives; do not invent one. Set `status: draft`.
6. **Commit:** `change(<customer>): draft <slug>`.

## Phase 2 — Get it reviewed

Review happens in git, by people. The skill helps it move; it does not decide, approve, or gate. The default path is **local review**; the central CAB is the exception.

1. **Local review first.** Most changes are reviewed and approved locally, by the team that owns the work and will run it: the director + lead engineer for the area, plus a Bar Raiser for higher-risk changes. A Bar Raiser is an operationally experienced, trusted person, independent of the team, trained to hold the standard. It's a role, not a job title. Tell the user who the local review needs; they pick the people. Set `status: in-review` and commit.
2. **Escalate to the CAB only by exception.** A change goes to the central CAB when it's **high-risk** (e.g. prod + a critical customer, or shared infrastructure), **crosses teams**, or the local pair doesn't feel confident to own it. If the user judges it needs the CAB, note that in the record, and *why* (high-risk, or the local pair wasn't confident; that distinction is a useful signal). Otherwise, local approval is the approval.
3. **Help notify.** Offer to draft a short ready-for-review message (title, what the change does, customer/service/environment, risk in plain words, and the record's path) for the user to post to their review channel (Teams/Slack/email), and/or just confirm the record is committed so reviewers can read it in git.
4. **Record decisions.** As reviewers respond, capture each in the record's Review table: name, role (peer / lead / Bar Raiser / CAB), decision, date. The skill writes what the user relays; it does not chase or assign anyone.
5. **Mark approved.** When the required reviewers have approved, set `status: approved` (note in the record whether it was **locally approved** or **CAB-approved**) and commit: `change(<customer>): approved <slug>`.
6. **If the record is edited after review starts**, reset `status: draft`, clear the approvals, note it, and review starts again.

Do not proceed to execution until the record shows `status: approved`.

## Phase 3 — Execute it (read-do, closed loop)

Only for a record that is `approved` (locally or by CAB). Set `status: executing` and commit before the first step.

**Readiness checks first (go–no-go).** Before any change step, run the readiness checks (section 2a) the same read-do way: present each, the operator verifies it and pastes back what they saw, you record go or no-go. If **any check is no-go, stop**: the conditions to start aren't met. Do not begin the change steps. Surface it to the operator, who fixes the condition and re-checks, or aborts the window. Only when every readiness check is go do you move to the change steps.

Then work the change steps **one step at a time**. For each step:

1. **Present** the step: description, the command, the expected outcome, and the rollback if it fails. **Always put the command in its own fenced code block, on its own line, by itself,** so the operator gets a one-click copy and never has to hand-select text (selecting risks dropping a character mid-change). Never give a runnable command as inline text. Put the rollback command in its own fenced block too. If a step has several commands, give each its own block.
2. **For a destructive step or a hold point, ask the operator to confirm before running.** A deliberate pause at the riskiest moments.
3. The operator runs it in their own terminal and **pastes the actual output back**. Ask for the real output, not "done."
4. **Record** the observed output in the execution table. Don't ask the operator to log who ran it or when; the executor is named in the record header and the git commit timestamps it. At a hold point, do record who did the second-person verification, since that's a different person and new information not already in the record.
5. **Compare** the output to the expected outcome. If it matches, move on. If it does **not** match, stop and put the choice to the operator: continue (with a recorded reason), pause, or roll back. Record the decision and the reason. Never skip ahead or batch steps.
6. **Hold points:** after the operator's step, stop. A second person must confirm before the next step unlocks. Record who confirmed, in the Observed cell.

### When it stops being a change and becomes an incident

A failed step and a live outage are not the same thing, and the skill must not treat them the same. Watch for the operator signalling real impact: "the system is down," "customers are affected," "it's throwing errors for everyone," panic. The moment that happens, say so plainly: **this is now an incident, not a change.** Then:

1. **Restoring service comes first, not the record.** If the rollback is safe, do it now (present the rollback command in its own block). Don't slow the operator down with bookkeeping while production is down.
2. **Tell them to escalate / declare per their incident process.** This is no longer something the change skill owns. The change record hands off to incident response (a different team, a different process).
3. **Once service is back and there's breathing room**, capture what happened in the record (the step that failed, the impact, the rollback, the timeline), because that record is now incident input too.
4. The change is automatically a divergence → it will go to review-back (Phase 5), and given customer impact it almost certainly warrants a postmortem, not just a template fix. Flag that, but the severity call is the reviewers'/CAB's, not the skill's.

Stay calm and concrete. The operator is stressed; one instruction at a time, the rollback command ready to copy.

### When the operator adapts (this is expected, not a failure)

The plan rarely survives contact with the system. A command fails, a step is missing, the operator improvises. Capture it, don't fight it.

- **Changed step** (they altered a planned command on the fly): record what they actually ran, set that step's origin to `changed`, and capture the reasoning: expected what, got what, did what instead and why.
- **Added step** (a new step not in the plan): add it to the execution table with origin `added`. Before it runs, do a **micro pre-flight**, three quick questions: *what does this do? what happens if it fails? how do you undo it?* Record the answers. Then run it read-do like any other step.
- **Capture the why, always.** The rationale is the most valuable thing here; it's what the later review learns from.
- **Material divergence:** if the operator is doing something well outside what the CAB approved, say so plainly. Pause, surface it: this is now meaningfully different from what was signed off. The operator decides: continue with the reasoning recorded, or stop and get eyes on it. Show, don't decide.

Commit when execution completes: `change(<customer>): executed <slug>`.

## Phase 4 — Verify it

The mirror of the readiness checks. Walk the verification checklist the same read-do way: present check, operator runs it, pastes the result, you record it. Confirm two things, not just that the change ran:

- **The system is back to normal and healthy:** re-run the readiness checks that mattered (health good, version now as intended, maintenance mode lifted, nothing left degraded).
- **The customer can use the service again:** confirmed from the customer's entry point, not just the operator's view (for example, a real customer action succeeds).

If a verification check fails, treat it like a mismatch in execution: stop, surface it, the operator decides (continue with reason / pause / roll back). Record observed results. Set `status: verified` and commit.

## Phase 5 — Close the loop

Verified is not automatically closed. Decide the close state from what actually happened during execution; the skill already knows, because it recorded the steps.

- **Went to plan** (every step `planned`, every output matched, no roll-back): set `status: closed`. Commit: `change(<customer>): closed <slug>`.
- **Diverged** (any `changed` or `added` step, any failed step, any mismatch the operator continued past): set `status: flagged for review-back`. This is not a verdict; it means the plan and reality came apart, and that gap is information the reviewers need. Review-back goes to whoever owns the change (local review, or the CAB if it was an escalated change).

For a diverged change, **fill the Close-out section** of the record: went-to-plan = no, what diverged (the changed/added/failed steps and their reasoning), and leave the outcome line blank. Present the divergence to the user and pose the graduated question for the review: is this a template fix, a lesson worth circulating, or a postmortem? **Do not decide severity;** that's the reviewers' call; the skill makes sure they have the material.

The record stays `flagged for review-back` until the outcome is recorded back in it: "reviewed on <date> by <local / CAB>; outcome: template updated / lesson logged / postmortem opened (link)." Only then set `status: closed`. That visible open state is deliberate: it stops the lesson from being acknowledged and quietly dropped.

If a divergence (or the review-back) reveals the review should have asked something it didn't, offer to update `reference/preflight-questions.md` or `template.md` so the next change asks better. That's the process learning from its own misses.

## After: save a template

If the change went well and will recur, offer to save it as a reusable starting point (strip customer-specific detail; the procedure transfers, the context doesn't).

## Principles

- Built for manual changes. The execution phase (one step at a time, hold points, capturing what was actually observed and how the operator adapted) is the heart of it.
- The pre-flight is the point. If the user wants to skip it, that's the signal it matters most.
- Show, don't decide. The skill records, guides, surfaces, and asks. People approve, judge severity, and choose how to handle a mismatch.
- The git history is the audit trail and the accountability record: named author, named reviewers, the executor in the header, what was observed at each step. Ownership and traceability, not blame.
- Never run the change. The operator runs every command; the skill keeps them thinking before they act and on track while they act.
- Don't burden the operator with bookkeeping a tool can do. The skill fills in what it knows (the executor from the header, timing from git) and asks the operator only for what carries real information: the observed output, the reasoning behind an adaptation, the second person at a hold point.
- Capture reasoning, not just answers. The hedges and assumptions surfaced in conversation go into "Reasoning & open assumptions," curated, not a raw transcript. (A verbatim transcript can bury the signal and chill candor; if a team wants maximum fidelity they can opt into keeping the raw conversation alongside the record, but the curated reasoning trail is the default.)
