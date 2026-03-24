# TrilogyOS — Admissibility Model
# Version: 0.1.0 (Surface Release)
# Status: PUBLIC / NON_EXEC
# Class: VERIFICATION SURFACE ONLY

---

## Purpose

This document defines the admissibility evaluation model
used by TrilogyOS to determine whether an action is permitted.

It describes observable behaviour.
It does not describe enforcement implementation.

---

## Core Function

The system evaluates every action through a governance function:

    G(s, a) → {ALLOW, DENY, HOLD}

Where:
- `s` = current system state
- `a` = proposed action
- Output = one of three verdicts (closed set, no other values exist)

---

## Verdict Definitions

### ALLOW
The action is admissible in the current state.
Execution may proceed.

### DENY
The action is not admissible in the current state.
Execution is blocked. No partial execution. No retry without state change.

### HOLD
The action cannot be evaluated with available information.
Execution is suspended until the missing condition is resolved.
HOLD is not a soft ALLOW. Nothing proceeds under HOLD.

---

## Evaluation Properties

### 1. Pre-Execution Only

G(s, a) is evaluated before any state change occurs.
Post-hoc evaluation is not admissibility — it is audit.

### 2. Closed-World Assumption

The system operates under a closed-world assumption:
- Only defined actions are evaluated
- Undefined actions are not "probably fine"
- Undefined actions → DENY

### 3. Determinism

Given the same state `s` and the same action `a`,
the same verdict must be reachable.

If the system produces different verdicts for identical inputs
without a documented state change, it is drifting.

### 4. No Optimistic Interpretation

Ambiguity in the action or state does not resolve toward ALLOW.
Ambiguity resolves toward HOLD or DENY.

The system does not guess that something is probably safe.

---

## Authority Model

### Recomputed at Every Evaluation

Authority is never cached from a previous cycle.
Each evaluation traces authority to origin or to an explicit bounded instruction.

Stale authority = no authority.

### Origin

All authority traces back to a human origin layer.
No internal component may generate authority independently.

---

## Commit Boundary

A commit boundary is any point where:
- State would change
- An external effect would occur
- A decision would become irreversible

At every commit boundary, the governance function must be called.

If a commit-like action is implicated and the governance function is not called,
the action is invalid regardless of outcome.

---

## What This Model Does Not Describe

- How the governance function is implemented
- What heuristics, weights, or scoring are used internally
- How the system routes actions to the governance layer
- How enforcement is applied after a verdict
- Internal optimisation strategies

These are enforcement details and are not part of this surface.
