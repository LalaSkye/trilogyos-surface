# TrilogyOS — Verification Surface v0.1

**A public, non-executing constraint model for admissibility-first governance.**

---

## What This Is

This repository contains the verification surface for TrilogyOS —
a constraint-first operating protocol for human–AI systems.

It publishes:
- The admissibility evaluation model
- State transition rules
- Fail-closed semantics
- Worked examples (ALLOW, DENY, HOLD decisions)
- An adversarial test harness (10 attack scenarios)
- Sample decision logs
- The system's constraint invariants

## What This Is Not

This is not the system itself.

This repository does not contain:
- The enforcement engine
- Internal routing or orchestration logic
- Heuristics, scoring, or weighting
- Prompt structures or optimisation strategies
- Anything that enables reproduction of execution behaviour

**If it explains behaviour → it's here.
If it enables replication → it's not.**

---

## Repository Structure

```
trilogyos-surface-v0.1/
├── README.md
├── LICENSE
├── spec/
│   ├── admissibility_model.md      — G(s,a) → {ALLOW, DENY, HOLD}
│   ├── state_transition_rules.md   — Transition gating and rules
│   └── fail_closed_semantics.md    — Fail-closed default table and packet rules
├── examples/
│   ├── allowed_transition.json     — Valid action, valid state → ALLOW
│   ├── denied_transition.json      — Scope escalation without authority → DENY
│   ├── held_transition.json        — Missing state field → HOLD
│   └── unknown_action.json         — Undefined action type → DENY
├── harness/
│   └── adversarial_tests.md        — 10 adversarial scenarios with expected outcomes
├── logs/
│   └── sample_decision_logs.json   — 12 sample decisions (4 ALLOW, 6 DENY, 2 HOLD)
├── principles/
│   └── constraint_invariants.md    — 7 system invariants
└── LICENSE                         — All rights reserved, verification use only
```

---

## The Constraint Model

The system evaluates every action through a governance function:

```
G(s, a) → {ALLOW, DENY, HOLD}
```

Where:
- `s` = current system state
- `a` = proposed action
- The verdict set is closed. There is no fourth value.

### Core Properties

| Property                  | Behaviour                                        |
|---------------------------|--------------------------------------------------|
| Unknown action            | → DENY                                           |
| Missing state field       | → HOLD                                           |
| Ambiguous scope           | → HOLD                                           |
| Ungated commit boundary   | → Invalid (always)                               |
| Stale authority           | → No authority                                   |
| HOLD timeout              | → HOLD persists (does not expire into ALLOW)     |
| Refusal                   | → Valid system output, not failure               |

---

## The Invariants

1. **Fail-closed** — Unknown → DENY, missing → HOLD, ambiguous → HOLD
2. **Commit-boundary enforcement** — No state change without governance evaluation
3. **Authority recomputation** — Authority is never cached from a previous cycle
4. **Non-interpretation** — Controlled terms bind only inside structured envelopes
5. **Determinism** — Same input + same state = same verdict
6. **Refusal is valid** — DENY and HOLD are legitimate outputs, not errors
7. **No role collapse** — No layer acquires the powers of another

See [principles/constraint_invariants.md](principles/constraint_invariants.md) for detail.

---

## Adversarial Surface

The [harness](harness/adversarial_tests.md) documents 10 attack scenarios:

| Attack Class              | Expected Verdict |
|---------------------------|------------------|
| Governance bypass         | DENY             |
| Authority replay          | DENY             |
| Schema violation          | HOLD             |
| Undefined input injection | DENY             |
| Scope escalation          | DENY             |
| Role impersonation        | DENY             |
| Schema injection          | REJECT/STRIP     |
| Ambiguity exploitation    | HOLD             |
| Verdict injection         | REJECT           |
| Temporal bypass           | HOLD (persists)  |

---

## Design Posture

- Engineering, not promotional
- No claims about intelligence, cognition, or performance
- Show, don't argue
- The system denies more than it allows — this is by design

---

## Verification Goal

A competent engineer should be able to:
- Understand the constraint model
- Predict system decisions from the published rules
- Verify fail-closed behaviour against the examples

A competent engineer should not be able to:
- Reproduce the system
- Infer enforcement logic
- Bypass the constraint surface

---

## License

Copyright (c) 2026 Os-Trilogy LMT. All rights reserved.

Published for verification and review purposes only.
No commercial use. No redistribution. No derivative works.
See [LICENSE](LICENSE) for full terms.

---

## Author

Ricky Jones
Os-Trilogy LMT
