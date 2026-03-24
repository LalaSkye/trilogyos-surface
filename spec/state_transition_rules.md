# TrilogyOS — State Transition Rules
# Version: 0.1.0 (Surface Release)
# Status: PUBLIC / NON_EXEC
# Class: VERIFICATION SURFACE ONLY

---

## Purpose

This document defines the rules governing state transitions in TrilogyOS.

A state transition is any change from one system state to another.
Every transition must pass through the governance function before execution.

---

## Transition Model

A state transition has the form:

    s₁ --[a]--> s₂

Where:
- `s₁` = current state
- `a`  = proposed action
- `s₂` = resulting state (only reached if G(s₁, a) = ALLOW)

---

## Rules

### Rule 1 — No Ungated Transition

No state transition may occur without governance evaluation.

    if G(s, a) was not called:
        transition is invalid regardless of outcome

A successful outcome does not retroactively validate an ungated transition.

### Rule 2 — Verdict Precedes Execution

The governance verdict must be produced before the action executes.
Concurrent evaluation and execution is not permitted.

    evaluate → then act (sequential, never parallel)

### Rule 3 — DENY Blocks Completely

If G(s, a) = DENY:
- No partial execution
- No side effects
- No state change
- System remains in s₁

DENY is terminal for that action in that state.

### Rule 4 — HOLD Suspends

If G(s, a) = HOLD:
- Execution is suspended
- System remains in s₁
- Resolution requires new information or human input
- HOLD does not expire into ALLOW

### Rule 5 — ALLOW Does Not Persist

An ALLOW verdict is valid for one transition only.
It does not carry forward to subsequent actions.

Each new action requires a new evaluation.

### Rule 6 — Unknown Actions

If the action `a` is not in the defined action set:

    G(s, a) = DENY

Unknown actions are never permitted.
The system does not infer intent from undefined inputs.

### Rule 7 — Missing State Fields

If the state `s` is missing required fields:

    G(s, a) = HOLD

The system does not evaluate with incomplete state.
It waits for the missing information.

### Rule 8 — Ambiguous Scope

If the scope of the action is ambiguous:

    G(s, a) = HOLD

Ambiguity does not resolve toward ALLOW.

---

## Transition Table (Summary)

| Condition                      | Verdict | System Response          |
|--------------------------------|---------|--------------------------|
| Valid state, valid action      | ALLOW   | Transition proceeds      |
| Valid state, denied action     | DENY    | No transition, blocked   |
| Incomplete state               | HOLD    | Suspended, awaiting info |
| Unknown action                 | DENY    | Rejected outright        |
| Ambiguous scope                | HOLD    | Suspended, awaiting info |
| Commit boundary, no evaluation | —       | Invalid (always)         |

---

## What These Rules Do Not Describe

- The internal state representation
- How states are composed or decomposed
- Transition priority or ordering heuristics
- Internal routing between system components
- Recovery or rollback mechanisms

These are implementation details and are not part of this surface.
