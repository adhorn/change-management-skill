<!--
A filled-in change record, for reference. It shows the level of detail a good record carries:
each execution step says what to run, what to expect, and what to do if it's wrong.
Customer/service names here are placeholders. This is a "went to plan" example;
a diverged change would also fill in the Close-out section with what came apart.
-->

---
title: Increase Postgres connection limit (customer-x / prod-eu)
customer: customer-x
service: order-search
environment: prod-eu
author: jordan.lee
executor: jordan.lee
tags: [database]
status: closed
cloned_from:
---

# Change: Increase Postgres connection limit (customer-x / prod-eu)

**Status:** draft → in-review → approved → executing → verified → closed
**Approval route:** local

## Phase 1 — Pre-flight (the thinking)

### The change
**What is this change?**
> Raise `max_connections` on the customer's primary Postgres instance from 100 to 150.

**What is the expected outcome?**
> The instance accepts up to 150 concurrent connections, and the connection-timeout errors the customer hits during their nightly batch stop.

### What the customer experiences
**Will the customer notice this change?**
> Not directly; the change is at the database layer. They notice the timeout errors stopping.

**How will they notice it?**
> Fewer failed jobs in their nightly batch run.

**If this change fails mid-way, what is the customer in the middle of doing?**
> Nothing live at the chosen window (02:00 UTC, outside their business hours and batch windows).

**What happens to their in-progress work if it fails?**
> Nothing in progress at that time; worst case the restart is delayed.

### Failure and recovery
**What happens if this change fails?**
> If the parameter is wrong or the instance won't restart, the customer's app can't open new DB connections and is effectively down until fixed.

**How do you roll back?**
> Set `max_connections` back to 100 and restart (same procedure as the change, old value).

**How long does rollback take?**
> About 5 minutes: one config change plus one restart.

**What does the customer experience during rollback?**
> The same brief connection drop as the change itself, at a time they aren't active.

**What is the blast radius?**
> One customer, one instance, not shared infrastructure. If it goes wrong only this customer is affected, but for them it's a full outage until restored.

### Timing and coordination
**Is there a maintenance window, and when?**
> Yes, Saturday 02:00–03:00 UTC.

**Is this the lowest-impact window for the customer, or the most convenient for the operator?**
> Lowest-impact for the customer: outside business hours, after their batch completes.

**Are there dependencies on other changes or teams?**
> None.

### Customer awareness and agreement
**Is the customer aware this change is happening?**
> Yes, the account manager notified them five days ago.

**Has the customer agreed to it?**
> Yes, confirmed by the account manager.

**Has a maintenance window been communicated to the customer?**
> Yes, the Saturday 02:00 UTC window was sent.

**Who is the customer contact during this change?**
> The account manager (out of hours: the on-call account line).

**How will the customer be notified when it's complete?**
> Email from the account manager.

### Reasoning & open assumptions
Done this exact change on two other instances; confident in the steps and the rollback. Assuming 150 is within the instance's memory headroom; checked against current usage, it is. No snapshot needed since the change is a parameter value, fully reversible.

## Phase 2 — Execution (the doing)

### 2a — Readiness checks (pre-flight / go–no-go)

| # | Check | How to verify | Expected (go) | Observed | Go / no-go |
|---|---|---|---|---|---|
| 1 | Instance healthy before starting | `SELECT 1;` against the primary | Returns `1`, responds immediately | Returned `1` immediately | go |
| 2 | Current value is the expected baseline | `SHOW max_connections;` | Returns `100` | Returned `100` | go |
| 3 | In window and customer not active | Confirm time is in window; `SELECT count(*) FROM pg_stat_activity WHERE state = 'active';` | In window; near-zero active sessions | 02:05 UTC, 0 active sessions | go |

### 2b — Change steps

| # | Step | Origin | Command | Expected | Rollback | Observed |
|---|---|---|---|---|---|---|
| 1 | Set the new value | planned | `ALTER SYSTEM SET max_connections = 150;` | `ALTER SYSTEM` returned, no error | `ALTER SYSTEM SET max_connections = 100;` | `ALTER SYSTEM`, no error |
| 2 | Restart the instance to apply 🔒 | planned | `sudo systemctl restart postgresql@cust-x-prod` | Service returns to `active (running)` within 60s | Restore value (step 1 rollback), restart again | `active (running)` after ~12s. Hold point verified by Priya Shah |
| 3 | Confirm the new value is live | planned | `SHOW max_connections;` | Returns `150` | If still `100`, restart didn't pick up the change; investigate before continuing | Returned `150` |

## Phase 3 — Verification (the proving)

| # | Check | Side (system / customer) | Expected | Observed |
|---|---|---|---|---|
| 1 | New limit live | system | `SHOW max_connections;` returns `150` | Returned `150` |
| 2 | Instance accepting connections | system | `SELECT 1;` returns `1` | Returned `1` |
| 3 | Customer's batch runs clean | customer | Next nightly batch completes with no connection-timeout errors | Batch completed, zero timeout errors (30+ the night before) |

## Review

| Reviewer | Role (peer / lead / Bar Raiser / CAB) | Decision | Comments | Date |
|---|---|---|---|---|
| Tom Lee | peer | approved | Steps clear, rollback is sound | 2026-05-27 |
| Priya Shah | Bar Raiser | approved | Happy with the window and the memory check | 2026-05-27 |

## Close-out

- **Went to plan?** yes
- **If no, what diverged:** n/a
- **Review-back outcome:** n/a (no divergence)
