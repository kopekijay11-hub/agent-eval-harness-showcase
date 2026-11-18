# Task taxonomy (task types)

This page describes the **types** of tasks used for evaluation at a high level. It intentionally omits task texts/prompts, concrete instances, and battery contents.

## Task type categories

| Type | What it exercises (level of behavior) | Typical evidence in a trace |
|---|---|---|
| **Tool-selection & sequencing** | Choosing appropriate tools and invoking them in a required order to satisfy a goal. | Presence, identity, and ordering of tool calls relative to expectations. |
| **Argument & parameter conformance** | Supplying tool arguments that satisfy stated constraints/boundaries (shape, values, or required fields) without over- or under-specifying. | Canonicalized call arguments/signatures compared against expected constraints. |
| **Grounded read-then-act** | Consulting available context or artifacts via tools before taking a dependent action or stating a conclusion. | Read/inspection calls preceding dependent write/action calls or conclusions. |
| **State/constraint adherence** | Respecting explicit rules, limits, or invariants across steps (including boundary conditions) while completing the task. | Tool-call sequence and arguments consistent with declared constraints; absence of disallowed actions. |
| **Multi-step completion** | Carrying a short workflow to a verifiable end state using multiple coordinated actions. | Complete, ordered set of required calls and a resulting state consistent with success criteria. |

## How types relate to grading

- Tasks are graded on recorded tool-call behavior (identity, order, canonicalized arguments) and outcome criteria defined per task type, not on final prose alone.
- Outcomes are reported as pass/fail brackets/counts; type-level descriptions here do not reveal task texts, prompts, or scoring internals.
