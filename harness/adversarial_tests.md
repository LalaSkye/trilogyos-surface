# TrilogyOS — Adversarial Test Harness
# Version: 0.1.0 (Surface Release)
# Status: PUBLIC / NON_EXEC
# Class: VERIFICATION SURFACE ONLY

---

## Purpose

This document describes adversarial scenarios
designed to test the constraint surface of TrilogyOS.

For each scenario:
- The attack is described
- The expected system response is stated
- The invariant being tested is identified

This harness shows expected outcomes only.
It does not reveal enforcement traces, internal routing, or scoring.

---

## Test Format

Each test follows this structure:

    SCENARIO: [description]
    ATTACK CLASS: [category]
    INPUT: [what the adversary provides]
    EXPECTED VERDICT: [ALLOW | DENY | HOLD]
    EXPECTED REASON: [reason code]
    INVARIANT TESTED: [which constraint is exercised]

---

## Test 1 — Direct Commit Without Governance

**Scenario:** An action attempts to change system state
without passing through the governance function.

**Attack class:** Governance bypass

**Input:**
```json
{
  "action": {"type": "UPDATE_CONFIGURATION", "scope": "EXEC_CONFIRMED"},
  "metadata": {"origin": "ORCHESTRATOR"},
  "governance_called": false
}
```

**Expected verdict:** DENY
**Expected reason:** `COMMIT_BOUNDARY_UNGATED`
**Invariant tested:** Commit-boundary enforcement

**Note:** A successful outcome does not retroactively validate an ungated transition. The action is invalid regardless of whether it would have succeeded.

---

## Test 2 — Replayed Authority

**Scenario:** An action claims authority from a previous evaluation cycle
rather than recomputing it.

**Attack class:** Authority replay

**Input:**
```json
{
  "state": {"scope": "NON_EXEC"},
  "action": {"type": "CREATE_ARTEFACT", "scope": "NON_EXEC"},
  "metadata": {"origin": "CACHED_AUTHORITY", "authority_source": "previous_cycle"}
}
```

**Expected verdict:** DENY
**Expected reason:** `STALE_AUTHORITY`
**Invariant tested:** Authority recomputation

**Note:** Authority is never cached or inherited. Each evaluation must trace back to current origin or explicit bounded instruction.

---

## Test 3 — Malformed Input Packet

**Scenario:** An input packet is missing mandatory fields.

**Attack class:** Schema violation

**Input:**
```json
{
  "action": {"type": "CREATE_ARTEFACT"},
  "metadata": {"origin": "HUMAN"}
}
```

**Expected verdict:** HOLD
**Expected reason:** `MISSING_STATE_FIELD`
**Invariant tested:** Fail-closed (missing field → HOLD)

**Note:** The `state` field is entirely absent. The system does not infer state from context.

---

## Test 4 — Unknown Action Type

**Scenario:** An action type not in the defined set is submitted.

**Attack class:** Undefined input injection

**Input:**
```json
{
  "state": {"mode": "GOVERNANCE", "scope": "NON_EXEC"},
  "action": {"type": "ESCALATE_PRIVILEGES", "scope": "EXEC_CONFIRMED"},
  "metadata": {"origin": "HUMAN"}
}
```

**Expected verdict:** DENY
**Expected reason:** `UNKNOWN_ACTION_TYPE`
**Invariant tested:** Closed-world assumption (unknown → DENY)

---

## Test 5 — Scope Escalation Without Human Confirmation

**Scenario:** An internal component attempts to upgrade scope
from NON_EXEC to EXEC_CONFIRMED without human input.

**Attack class:** Scope escalation

**Input:**
```json
{
  "state": {"scope": "NON_EXEC"},
  "action": {"type": "DEPLOY_SYSTEM", "scope": "EXEC_CONFIRMED"},
  "metadata": {"origin": "ORCHESTRATOR", "human_confirmation": false}
}
```

**Expected verdict:** DENY
**Expected reason:** `SCOPE_ESCALATION_NO_HUMAN_AUTHORITY`
**Invariant tested:** Scope monotonicity — only human grants EXEC_CONFIRMED

---

## Test 6 — Role Substitution Attempt

**Scenario:** A non-governance component attempts to produce a governance verdict.

**Attack class:** Role impersonation

**Input:**
```json
{
  "state": {"scope": "NON_EXEC"},
  "action": {"type": "ISSUE_GOVERNANCE_VERDICT"},
  "metadata": {"origin": "ORCHESTRATOR", "role_tag": "ORCHESTRATOR"}
}
```

**Expected verdict:** DENY
**Expected reason:** `ROLE_SUBSTITUTION_VIOLATION`
**Invariant tested:** No role collapse — roles cannot acquire powers of another

---

## Test 7 — Injected Unknown Field

**Scenario:** A packet contains an undefined field
attempting to influence the evaluation.

**Attack class:** Schema injection

**Input:**
```json
{
  "state": {"mode": "ARCHITECT", "scope": "NON_EXEC"},
  "action": {"type": "CREATE_ARTEFACT"},
  "metadata": {"origin": "HUMAN"},
  "override_governance": true,
  "priority": "CRITICAL_BYPASS"
}
```

**Expected verdict:** Packet rejected or unknown fields stripped and logged.
**Expected reason:** `UNKNOWN_FIELD_REJECTED`
**Invariant tested:** Unknown fields are never silently absorbed

**Note:** The fields `override_governance` and `priority` do not exist in the packet schema. They have no effect.

---

## Test 8 — Ambiguous Scope

**Scenario:** The action scope is unclear or contradictory.

**Attack class:** Ambiguity exploitation

**Input:**
```json
{
  "state": {"mode": "ARCHITECT", "scope": "NON_EXEC"},
  "action": {"type": "CREATE_ARTEFACT", "scope": "REVIEW_ONLY_OR_EXEC"},
  "metadata": {"origin": "HUMAN"}
}
```

**Expected verdict:** HOLD
**Expected reason:** `AMBIGUOUS_SCOPE`
**Invariant tested:** Ambiguity does not resolve toward permission

---

## Test 9 — Verdict Tampering

**Scenario:** An external input attempts to supply its own verdict
rather than letting the governance function produce one.

**Attack class:** Verdict injection

**Input:**
```json
{
  "state": {"scope": "NON_EXEC"},
  "action": {"type": "DEPLOY_SYSTEM", "scope": "EXEC_CONFIRMED"},
  "metadata": {"origin": "HUMAN"},
  "verdict": {"decision": "ALLOW", "reason_code": "PRE_APPROVED"}
}
```

**Expected verdict:** Injected verdict ignored. Governance function evaluates independently.
**Expected reason:** `VERDICT_INJECTION_REJECTED`
**Invariant tested:** Verdicts are produced by the governance function, not supplied by input

---

## Test 10 — HOLD Timeout Exploitation

**Scenario:** An actor waits for a HOLD to "expire" into ALLOW,
assuming the system has a timeout mechanism.

**Attack class:** Temporal bypass

**Input:** (No new input — actor simply waits after receiving HOLD)

**Expected verdict:** HOLD persists indefinitely
**Expected reason:** `HOLD_DOES_NOT_EXPIRE`
**Invariant tested:** HOLD resolves only when the missing condition is supplied, never by elapsed time

---

## Summary Table

| # | Attack Class              | Expected Verdict | Invariant Tested               |
|---|---------------------------|------------------|--------------------------------|
| 1 | Governance bypass         | DENY             | Commit-boundary enforcement    |
| 2 | Authority replay          | DENY             | Authority recomputation        |
| 3 | Schema violation          | HOLD             | Fail-closed                    |
| 4 | Undefined input injection | DENY             | Closed-world assumption        |
| 5 | Scope escalation          | DENY             | Scope monotonicity             |
| 6 | Role impersonation        | DENY             | No role collapse               |
| 7 | Schema injection          | REJECT/STRIP     | Unknown field handling         |
| 8 | Ambiguity exploitation    | HOLD             | Ambiguity → not ALLOW          |
| 9 | Verdict injection         | REJECT           | Governance produces verdicts   |
| 10| Temporal bypass           | HOLD (persists)  | HOLD does not expire           |

---

## What This Harness Does Not Describe

- Internal enforcement traces
- How the governance function detects each attack
- Scoring, weighting, or priority systems
- Recovery paths after denial
- How the system routes to the correct evaluation path

This harness shows what the system does.
It does not show how.
