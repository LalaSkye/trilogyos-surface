# TrilogyOS — Constraint Invariants
# Version: 0.1.0 (Surface Release)
# Status: PUBLIC / NON_EXEC
# Class: VERIFICATION SURFACE ONLY

---

## Purpose

This document lists the constraint invariants
that govern TrilogyOS behaviour.

These are properties the system must always satisfy.
If any invariant is violated, the system is in an invalid state.

This document describes the invariants themselves.
It does not describe how they are enforced.

---

## Invariant 1 — Fail-Closed

The system defaults to denial or hold on ambiguity.

- Unknown action → DENY
- Missing field → HOLD
- Ambiguous scope → HOLD

No silent pass-through.
No optimistic interpretation.

---

## Invariant 2 — Commit-Boundary Enforcement

No state change occurs without the governance function evaluating the boundary.

If a commit-like action is implicated and governance is not called,
the action is invalid regardless of outcome.

---

## Invariant 3 — Authority Recomputation

Authority is never cached or inherited from a previous cycle.

Each decision must trace back to origin authority
or to an explicit bounded instruction.

Stale authority = no authority.

---

## Invariant 4 — Non-Interpretation

Controlled terms bind only inside structured envelopes.

- No synonym substitution
- No inferential expansion
- No decorative restatement of formal language

If a term is defined, it means what the definition says.
Nothing else.

---

## Invariant 5 — Determinism

Given the same input and the same system state,
the same verdict must be reachable.

If the system produces different outputs for identical inputs
without a documented state change, it is drifting.

---

## Invariant 6 — Refusal Is Valid

DENY and HOLD are legitimate system outputs.

Refusal is not failure.
Refusal is the system working correctly at a boundary.

No pressure — internal or external — may treat refusal
as an error to be overcome.

---

## Invariant 7 — No Role Collapse

No internal role may acquire the powers of another
without explicit bounded instruction.

The system is composed of separated functional layers.
Each layer performs its defined function and no other.

Role substitution = system failure.

---

## Invariant Properties

| #  | Name                      | Failure Mode if Violated              |
|----|---------------------------|---------------------------------------|
| 1  | Fail-closed               | Undefined input silently permitted    |
| 2  | Commit-boundary           | Ungated state changes                 |
| 3  | Authority recomputation   | Stale permissions persist             |
| 4  | Non-interpretation        | Controlled terms drift in meaning     |
| 5  | Determinism               | Same input, different verdicts        |
| 6  | Refusal is valid          | System pressured past its own limits  |
| 7  | No role collapse          | Governance lost through role blending |

---

## Relationship Between Invariants

These invariants are not independent — they reinforce each other:

- **1 + 2:** Fail-closed ensures ungated commits are caught.
- **2 + 3:** Commit-boundary enforcement requires fresh authority each time.
- **4 + 5:** Non-interpretation supports determinism (same terms → same meaning → same verdict).
- **6 + 7:** Refusal-as-valid prevents pressure toward role collapse.
- **1 + 6:** Fail-closed and refusal-as-valid together mean the system can stop without apology.

---

## What This Document Does Not Describe

- How invariants are monitored at runtime
- What enforcement mechanisms detect violations
- Internal escalation paths when an invariant is threatened
- How invariants interact with specific system components
- Recovery procedures after an invariant violation

The enforcement details are not part of this surface.
