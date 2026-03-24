# TrilogyOS — Fail-Closed Semantics
# Version: 0.1.0 (Surface Release)
# Status: PUBLIC / NON_EXEC
# Class: VERIFICATION SURFACE ONLY

---

## Purpose

This document defines what "fail-closed" means in TrilogyOS
and how the system behaves at every failure boundary.

---

## Definition

A fail-closed system defaults to denial or suspension when:
- information is missing
- conditions are ambiguous
- the action is undefined
- the evaluation cannot complete

The system never fails open.
The system never assumes safety.

---

## Fail-Closed Table

| Condition                           | Default Verdict | Rationale                                     |
|-------------------------------------|-----------------|-----------------------------------------------|
| Unknown action                      | DENY            | Undefined input is never permitted            |
| Missing required field              | HOLD            | Incomplete data prevents safe evaluation      |
| Ambiguous scope                     | HOLD            | Ambiguity does not resolve toward permission  |
| Commit boundary without evaluation  | DENY            | Ungated transitions are always invalid        |
| Governance layer unavailable        | DENY            | No evaluation capability = no permission      |
| Unrecognised packet field           | REJECT or STRIP | Unknown structure is never silently absorbed  |
| Freeform reason (not coded)         | HOLD            | Reasons must be finite and versioned          |

---

## What "Fail-Closed" Is Not

### Not Fail-Safe
Fail-safe means the system enters a safe state on failure.
Fail-closed means the system stops and denies by default.
These overlap but are not identical.

### Not Conservative Optimism
The system does not say "probably fine but let's be careful."
The system says "not evaluated, therefore not permitted."

### Not Recoverable by Retry
A DENY on an unknown action does not become ALLOW on retry
unless the action is added to the defined set (a versioned change).

A HOLD does not expire into ALLOW.
It resolves only when the missing condition is supplied.

---

## Packet-Level Enforcement

### Input Packets

Input packets must contain:
- `state` — current system/task state
- `action` — what is being requested
- `metadata` — origin, timestamp, role tags

If any mandatory field is absent → HOLD.
If an unknown field is present → reject the packet or strip the field and log the event.

### Verdict Packets

Verdict packets must contain:
- `verdict` — one of: ALLOW, HOLD, DENY (closed set)
- `reason_code` — finite, versioned identifier
- `timestamp` — when the verdict was produced

Freeform reason strings are not reason codes.
If the reason code set needs to grow, it requires a version increment.

---

## The Closed Set Invariant

The verdict set is:

    {ALLOW, DENY, HOLD}

There is no fourth value.
There is no "ALLOW_WITH_WARNING" or "SOFT_DENY" or "MAYBE".

If a condition does not clearly resolve to ALLOW,
it resolves to HOLD or DENY.

---

## What This Document Does Not Describe

- How the fail-closed logic is implemented
- Internal fallback routing
- Error handling beyond the governance boundary
- Recovery procedures after DENY or HOLD
- How the system determines which table row applies

These are enforcement details and are not part of this surface.
