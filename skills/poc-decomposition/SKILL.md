---
name: poc-decomposition
description: >
  Operationalises a three-level agentic workflow architecture (Vision -> PoC Phases -> Tasks).
  Use when working on any PoC that uses this structure: to validate a phase is well-formed,
  decompose it into tasks, update the task ledger after execution, aggregate results, and
  determine whether a phase has been met. Invoke when a conversation involves defining or
  executing a PoC phase, reviewing task outcomes, or deciding whether to advance to the
  next phase.
---

# PoC Decomposition Skill

This skill operationalises the three-level agentic workflow architecture documented in the
Orchestration KB page. It governs how a Claude instance works with Vision, PoC Phases, and
Tasks — and how it maintains the task ledger that makes the workflow auditable.

---

## The Three-Level Architecture

**Level 1 — Vision.** Directional and aspirational. Not directly agent-executable. Does not
change unless the fundamental direction changes. Do not decompose Vision; use it for
orientation only.

**Level 2 — PoC Phases.** Each phase is a strategic goal with a demonstrable outcome. Phases
are sequential. Each must follow the standard format before it can be decomposed.

Standard PoC phase format:
- **Capability** — what is being demonstrated
- **Success Criteria** — a testable outcome that can be evaluated pass/fail, not a
  description of what will be built
- **Inputs** — what the phase requires to run
- **Expected Outputs** — what a passing phase produces
- **Status** — not started / in progress / validated / failed

**Level 3 — Tasks.** Derived from a phase by a coordination layer. Each task has its own
goal expressed as a small, testable success criterion. Tasks are the unit of agent execution.

---

## Step 1 — Validate a PoC Phase

Before decomposing, check the phase has all required fields. Flag and stop if any are missing
or malformed.

Checks to run:
1. **Capability** — present and clearly scoped? Flag if vague (e.g. "explore X" with no
   bounded outcome).
2. **Success Criteria** — present AND testable pass/fail? Flag if it describes activity
   rather than outcome (e.g. "build a script" instead of "script returns X given Y input").
   This is the most common failure point. Do not proceed with decomposition until success
   criteria can be evaluated without ambiguity.
3. **Inputs** — listed? Flag if any required input is undefined.
4. **Expected Outputs** — listed and consistent with the success criteria?
5. **Status** — one of the four valid values?

Output of this step: either a **validation pass** (proceed to decomposition) or a
**validation report** listing each failed check with a one-sentence explanation and a
suggested fix. Do not produce partial decompositions for a failing phase.

---

## Step 2 — Decompose a Phase into Tasks

Take a validated phase and produce a **decomposition record**. This record is produced once
per phase and must be stored in the task ledger.

The decomposition record contains:
- **Phase reference** — which phase this decomposition is for
- **Decomposition reasoning** — why this breakdown was chosen; what alternatives were
  considered and rejected; any assumptions made
- **Task list** — each task as a structured entry (see below)

Each task entry:
- **Task ID** — a short identifier (e.g. T1, T2) unique within the phase
- **Goal** — a single testable success criterion (same standard as the phase success
  criteria, but scoped to this task)
- **Inputs** — what this task needs (may be inherited from the phase or produced by a
  prior task)
- **Expected output** — what a passing task produces
- **Dependencies** — prior task IDs this task requires (if any)
- **Status** — not started

Guidelines for good decomposition:
- Tasks should be independently executable where possible
- No task goal should be so broad that it restates the phase goal
- The aggregate of all task goals should be sufficient to meet the phase success criteria —
  if they are not, the decomposition is incomplete
- If a task depends on the output of a prior task, note that dependency explicitly

---

## Step 3 — Update the Task Ledger After Execution

After a task completes, record the outcome in the task ledger. Do not summarise or
interpret — record what was produced and whether it met the task goal.

Update fields per task:
- **Status** → validated or failed
- **Actual output** — what was produced (be specific: file names, values, test results)
- **Validation result** — did the actual output meet the task goal? State yes or no,
  then one sentence of evidence
- **Failure notes** (if failed) — what was attempted, what the blocker was, whether retry
  is warranted

If a task fails, do not automatically advance. Flag the failure and surface the options:
retry with a modified approach, redefine the task goal, or escalate to phase level.

---

## Step 4 — Aggregate Task Results to Phase Outcome

Once all tasks in a phase have a terminal status (validated or failed), aggregate:

1. Count validated vs failed tasks
2. For each failed task: does the failure affect the phase success criteria? A task can
   fail without blocking the phase if its output is not required for the phase outcome
   (e.g. an optional optimisation step). State this explicitly.
3. Produce a **phase outcome assessment**:
   - phase validated — all success-criteria-relevant tasks validated; phase success
     criteria is met
   - phase failed — one or more success-criteria-relevant tasks failed; phase success
     criteria is not met
   - phase blocked — tasks cannot be evaluated yet (dependency not resolved, input missing)

The phase outcome assessment must reference the phase success criteria directly. Do not
declare a phase validated without tracing from task outputs to the success criteria.

---

## Step 5 — Flag Phase Transition Readiness

When a phase reaches phase validated:
1. State clearly: "Phase [name/number] is validated. Ready to advance."
2. Summarise what was demonstrated (one paragraph — this becomes the phase history entry)
3. If a next phase exists: surface it and offer to run Step 1 (validation) on it
4. If no next phase exists: flag that the PoC is complete and summarise what the full
   sequence demonstrated

When a phase reaches phase failed:
1. State clearly: "Phase [name/number] failed."
2. Identify the blocking tasks and their failure notes
3. Offer options: redefine the phase, redefine the failing tasks, or abandon

---

## The Task Ledger

The task ledger is implementation-agnostic. It may be a Notion database, a flat file,
a structured log, a GitHub issue tracker, a Google Sheet, or any other persistent store.
What matters is that it captures:
- The decomposition record (once per phase)
- Task status, actual outputs, and validation results (updated after each task)
- Phase outcome assessments (once per phase, after aggregation)

The specific implementation is defined per project. When this skill is invoked, ask where
the task ledger is stored if it is not already established in context.
