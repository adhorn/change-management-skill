---
title: <title>
customer: <customer>
service: <service>
environment: <environment>
author: <author>
executor: <who will run it; same as author unless handed over>
tags: []
status: draft
cloned_from:
---

# Change: <title>

**Status:** draft → in-review → approved → executing → verified → closed
<!-- approved = locally approved (default) OR CAB-approved (escalated changes). If execution diverged: verified → flagged for review-back → closed (with outcome recorded). -->
**Approval route:** <local | CAB; and if CAB, why escalated: high-risk / cross-team / local pair not confident>

## Phase 1 — Pre-flight (the thinking)

### The change
**What is this change?**
> <answer>

**What is the expected outcome?**
> <answer>

### What the customer experiences
**Will the customer notice this change?**
> <answer>

**How will they notice it?**
> <answer>

**If this change fails mid-way, what is the customer in the middle of doing?**
> <answer>

**What happens to their in-progress work if it fails?**
> <answer>

### Failure and recovery
**What happens if this change fails?**
> <answer>

**How do you roll back?**
> <answer>

**How long does rollback take?**
> <answer>

**What does the customer experience during rollback?**
> <answer>

**What is the blast radius?**
> <answer>

### Timing and coordination
**Is there a maintenance window, and when?**
> <answer>

**Is this the lowest-impact window for the customer, or the most convenient for the operator?**
> <answer>

**Are there dependencies on other changes or teams?**
> <answer>

### Customer awareness and agreement
**Is the customer aware this change is happening?**
> <answer>

**Has the customer agreed to it?**
> <answer>

**Has a maintenance window been communicated to the customer?**
> <answer>

**Who is the customer contact during this change?**
> <answer>

**How will the customer be notified when it's complete?**
> <answer>

### Reasoning & open assumptions
*What the operator was sure of, unsure of, and assuming when the plan was made, surfaced during write-up. The reviewer and CAB read this to weigh the risk; the learning loop reads it if the change later diverges. Capture hedges and uncertainties honestly: "no idea" and "not sure, the account manager would know" are valuable here, not failings.*

- <assumption or uncertainty, and who could confirm it, if anyone>

## Phase 2 — Execution (the doing)

### 2a — Readiness checks (pre-flight / go–no-go)

Before touching anything, confirm the conditions are right to start, like a pilot's pre-flight checklist. Each is something to verify, not something to change. If any check is no-go, you do not start the change.

| # | Check | How to verify | Expected (go) | Observed | Go / no-go |
|---|---|---|---|---|---|
| 1 |  |  |  |  |  |

### 2b — Change steps

The change itself. One action per step. Mark hold points 🔒. Origin: **planned** (reviewed before execution), **changed** (a planned step altered on the fly), **added** (new, not reviewed). For changed/added steps, put the reasoning in Observed: expected what, got what, did what instead, and why. For added steps, record the micro pre-flight (what it does / what if it fails / how to undo). At a hold point, record who did the second-person verification in Observed.

| # | Step | Origin | Command | Expected | Rollback | Observed (actual output; reasoning for changed/added; verifier for hold points) |
|---|---|---|---|---|---|---|
| 1 |  | planned |  |  |  |  |

## Phase 3 — Verification (the proving)

The mirror of the readiness checks: confirm the conditions are back to normal and healthy, and the customer can actually use the service again, not just that the change ran. Cover both sides:

- **System healthy / back to normal:** the readiness checks that mattered, re-run (system health good, version now as intended, maintenance mode lifted, nothing left degraded).
- **Customer can use the service:** confirmed from the customer's entry point, not just the operator's view (for example, a real customer action succeeds).

| # | Check | Side (system / customer) | Expected | Observed |
|---|---|---|---|---|
| 1 |  |  |  |  |

## Review

Local review by default (the team that owns the work, plus a Bar Raiser for higher-risk changes: an operationally experienced, trusted person, independent of the team, who holds the standard). Escalate to the CAB only if high-risk, cross-team, or the local pair isn't confident.

| Reviewer | Role (peer / lead / Bar Raiser / CAB) | Decision | Comments | Date |
|---|---|---|---|---|
|  |  |  |  |  |

All required reviewers approve before execution. Any edit after review starts resets the reviews (status drops to draft).

## Close-out

- **Went to plan?** <yes / no>
- **If no, what diverged:** <the changed / added / failed steps and the reasoning>
- **Review-back outcome:** <reviewed on [date] by [local / CAB]; outcome: template updated / lesson logged / postmortem opened [link]>

<!-- The change is not closed until the review-back outcome line is filled in for a diverged change. -->
