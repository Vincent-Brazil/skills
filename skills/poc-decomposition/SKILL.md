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

**Scope note:** the Orchestration architecture gives the coordination layer three
responsibilities: decomposition, execution routing, and validation. This skill covers
decomposition and validation. Execution routing — assigning tasks to agents with the right
capabilities — is a deliberate future direction, not yet operationalised; today, execution
is handled by whatever instance is running this skill.

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
- **Status** — not started / in progress / blocked / validated / failed

**Level 3 — Tasks.** Derived from a phase by a coordination layer. Each task has its own
goal expressed as a small, testable success criterion. Tasks are the unit of agent execution.

Task status values: not started / in progress / validated / failed / blocked. The terminal
statuses are validated, failed, and blocked — every task must reach one of these before
phase aggregation can run.

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
5. **Status** — one of the five valid values?

Output of this step: either a **validation pass** (proceed to decomposition) or a
**validation report** listing each failed check with a one-sentence explanation and a
suggested fix. Do not produce partial decompositions for a failing phase.

---

## Step 2 — Decompose a Phase into Tasks

Take a validated phase and produce a **decomposition record**. The record is versioned —
the first decomposition of a phase is v1 — and must be stored in the task ledger. See
"Re-decomposition" below for when later versions are created.

The decomposition record contains:
- **Phase reference** — which phase this decomposition is for
- **Version** — v1 for the initial decomposition
- **Decomposition reasoning** — why this breakdown was chosen; what alternatives were
  considered and rejected; any assumptions made
- **Task list** — each task as a structured entry (see below)

Each task entry:
- **Task ID** — a short identifier (e.g. T1, T2) unique within the phase, never reused
  across decomposition versions
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

### Re-decomposition

Execution often reveals that the initial breakdown was wrong or incomplete — a task needs
redefining, a missing task is discovered mid-phase, or the phase itself is revised. When
this happens, do not edit or delete the existing decomposition record. Create a new version
(v2, v3, …) in the ledger containing:
- **What changed** — tasks added, redefined, or withdrawn (withdrawn tasks keep their
  ledger history; mark them withdrawn rather than deleting)
- **Why** — reasoning that references the prior version and the execution evidence that
  prompted the change
- **Carried forward** — unchanged tasks keep their IDs, statuses, and recorded outcomes

New and redefined tasks get fresh IDs. Prior versions are never removed — the version
chain is what makes mid-phase changes auditable rather than silent.

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
retry with a modified approach, redefine the task goal (via re-decomposition), or escalate
to phase level.

**Failed dependencies:** when a task fails, every task that depends on it (directly or
transitively) can no longer run. Mark each such task **blocked** with a note naming the
failed dependency (e.g. "blocked: depends on T2, which failed"). This gives every task a
terminal status so aggregation can proceed — without it, the phase can never be assessed.
If the failed task is later retried successfully (or re-decomposition replaces it), reset
its dependents to not started.

---

## Step 4 — Aggregate Task Results to Phase Outcome

Once all tasks in the current decomposition version have a terminal status (validated,
failed, or blocked), aggregate:

1. Count validated vs failed vs blocked tasks
2. For each failed or blocked task: does it affect the phase success criteria? A task can
   fail without blocking the phase if its output is not required for the phase outcome
   (e.g. an optional optimisation step). State this explicitly.
3. Produce a **phase outcome assessment**:
   - phase validated — all success-criteria-relevant tasks validated; phase success
     criteria is met
   - phase failed — one or more success-criteria-relevant tasks failed (including tasks
     blocked by those failures); phase success criteria is not met
   - phase blocked — success-criteria-relevant tasks cannot be evaluated for reasons
     outside the decomposition (a phase input is missing, an external dependency is
     unresolved); the phase status is set to blocked until the blocker clears

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
3. Offer options: redefine the phase, redefine the failing tasks (both via
   re-decomposition — a new record version), or abandon

When a phase reaches phase blocked:
1. State clearly: "Phase [name/number] is blocked." Name the blocker.
2. Identify what would unblock it and who or what can act on that
3. Do not advance to the next phase past a blocked phase — phases are sequential

---

## The Task Ledger

The task ledger is implementation-agnostic. It may be a Notion database, a flat file,
a structured log, a GitHub issue tracker, a Google Sheet, or any other persistent store.
What matters is that it captures:
- The decomposition record (versioned; one chain per phase)
- Task status, actual outputs, and validation results (updated after each task)
- Phase outcome assessments (once per phase, after aggregation)

The specific implementation is defined per project. When this skill is invoked, ask where
the task ledger is stored if it is not already established in context.

---

## Platform Integrations (Optional)

Some platforms provide native session features that pair well with this skill. They are
conveniences layered on top of the ledger — the ledger remains the sole persistent record,
because native session features do not survive across sessions or platforms.

- **Session task trackers** (e.g. Claude Code's native task list): when executing a
  decomposed phase, mirror the task list into the tracker for live progress visibility.
  This is a display mirror only — every status change must still be written to the ledger,
  and on a new session the mirror is rebuilt from the ledger, never the reverse.
- **Goal-driven execution** (e.g. Claude Code's `/goal` command, which keeps the agent
  working until an independently evaluated completion condition is met): phase and task
  success criteria are written to be testable pass/fail, which is exactly the form such
  features need. These commands are user-invoked — the skill cannot set them — so when
  execution of a phase or task begins on a platform that supports this, suggest the user
  set the goal to the relevant success criterion verbatim.
- **Validation helpers** (e.g. Claude Code's `/verify`, `/run`, or `/code-review`
  commands, test suites, or equivalent): use these to generate the evidence recorded in
  Step 3 — a validation result should cite observed behaviour or test output, not the
  agent's assertion that the work is done. The evidence still goes in the ledger; the
  helper is just how it is produced.

On platforms without these features, skip this section — nothing in Steps 1–5 depends
on it.
