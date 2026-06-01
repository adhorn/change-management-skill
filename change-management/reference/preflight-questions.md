# Pre-flight questions

Ask these **one question at a time**: single question, wait for the answer, reflect it back in a line, then the next. Never present a section's questions as a list or batch. Where a line below reads as two questions ("Is there a maintenance window, and when?"), split it: ask the first, then ask the second in its own turn. Check each answer against the red flags; if one trips, push for a real answer or an explicit "not applicable, because…" (that push is its own turn too; don't also ask the next question). One question, one answer; don't let a compound answer skip the hard half.

---

## The change
*Framing: what are you doing and what should happen?*

**What is this change?**
What you are changing. Not why, just what.
- Good: "Increase the connection pool max_connections from 100 to 150 on the primary database."
- Red flag: vague verbs ("update", "fix", "tweak") with no object.

**What is the expected outcome?**
What should be different once this is applied. Specific and observable.
- Good: "The database accepts up to 150 concurrent connections; timeout errors during peak batch stop."
- Red flag: "it works" / "it's fixed" with nothing observable.

## What the customer experiences
*Framing: think from the customer's perspective, not yours.*

**Will the customer notice this change?** Yes/no and why: what the customer sees, not what you see.

**How will they notice it?** If they will, what they see or experience. If not, say so.

**If this change fails mid-way, what is the customer in the middle of doing?** Their workflow at the time.
- Red flag: "nothing" without considering batch windows, end-of-day processing, trading hours.

**What happens to their in-progress work if it fails?** Data loss? Retries? Blocked? Lost processing time?

## Failure and recovery
*Framing: assume this will go wrong. What then?*

**What happens if this change fails?** The failure scenario: what breaks, who's affected, what it looks like.

**How do you roll back?** The specific steps. Commands, not intentions.
- Good: "Set max_connections back to 100 in the parameter group; restart the connection pool; no data migration."
- Red flag: "revert it", "roll back the change", "restore from backup" with no steps. **Push hardest here:** this is the most common shallow answer, and the one that matters most under pressure.

**How long does rollback take?** Wall-clock from deciding to roll back to the customer being normal again.
- Red flag: no number.

**What does the customer experience during rollback?** From their side, what happens while you roll back.

**What is the blast radius?** How many customers, services, or environments are affected if this goes wrong.
- Red flag: "just this one" when the change touches shared infrastructure (a shared cluster, a shared DB host, shared networking). Check that explicitly: a change that looks small but sits on shared infra is the dangerous case.

## Timing and coordination
*Framing: is this the right moment, for the customer, not just for you?*

**Is there a maintenance window?** (Then, separately:) **When is it?** Specific date and time.

**Is this the lowest-impact window for the customer, or the most convenient for the operator?** Be honest.
- Red flag: a window chosen for operator convenience with no check of the customer's load.

**Are there dependencies on other changes or teams?** Does this depend on something first? Does something depend on this? Check shared systems and integrations (databases, data warehouses, message queues, auth services, batch schedules).

## Customer awareness and agreement
*Framing: does the customer know?*

**Is the customer aware this change is happening?**

**Has the customer agreed to it?** Agreement is not the same as awareness; did they explicitly approve?

**Has a maintenance window been communicated to the customer?** Did they get the date, time, and expected impact?

**Who is the customer contact during this change?** Name and how to reach them during execution.

**How will the customer be notified when it's complete?** The specific mechanism.

---

## Micro pre-flight (for steps added on the fly during execution)

When an operator adds a step that wasn't reviewed, don't run the full set. Ask three fast questions and record the answers, then run it read-do:

1. **What does this do?**
2. **What happens if it fails?**
3. **How do you undo it?**
