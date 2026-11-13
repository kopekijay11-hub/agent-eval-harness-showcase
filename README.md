# Agent Evaluation Harness — Showcase

**A standalone testing layer for AI coding agents — task batteries, tool-call grading, failure taxonomy, and regression detection across prompt and model changes.**

> This repository is a **showcase**. The working harness is private; what is documented here is the engineering: the trace format, the grading design, and the measured results. No live battery contents, real traces, or credentials are published.

---

## Public vs private

The boundary is deliberate, and it is the same one this project's safety rules were built on:

**Public (this repository):** the evaluation methodology · the trace format and its JSON Schema · the grading and regression design · summarized, verified results from the first live runs.

**Private (never published):** the working implementation and its full history · the live task battery (publishing it would train the subjects) · real recorded traces · operational detail.

## Project at a glance

| | |
|---|---|
| **Purpose** | Repeatable, evidence-grade evaluation of AI coding agents |
| **Grading unit** | The tool call — identity, order, and canonicalized arguments (RFC 8785 / JCS + SHA-256 signatures), not just the final answer |
| **Verdicts** | Multi-trial pass rates with drift statistics; a comparison with >25% invalid trials on either side is VOID, never averaged over |
| **Subjects** | Local models via Ollama and recorded agent traces — it runs free; no API key, no paid service |
| **Status** | Foundation, importer, grader, comparator, and replay complete · Runner 1 complete with first live grades recorded · 181-test suite, CI green |

## How to read this repository

- **README.md** (this file) — what the harness is and the results it stands on.
- **[docs/trace-format.md](docs/trace-format.md)** — the trace format in plain language: how a recorded agent run is represented.
- **[schema/trace.schema.json](schema/trace.schema.json)** — the JSON Schema (draft 2020-12) for one recorded run.
- **[examples/example_trace.json](examples/example_trace.json)** — a small hand-written trace that validates against the schema.
- **[docs/measured-results.md](docs/measured-results.md)** — the first live results, and what they actually establish.

---

## The four pillars

1. **Task batteries** — encoded tasks (prompt, expected conclusion, expected tool calls, failure labels) that an agent-under-test attempts.
2. **Tool-call grading** — traces record every tool call with identity (name + tool_call_id), order (monotonic sequence numbers), and arguments (raw arguments plus a canonicalized call signature). The grader compares live calls against expected calls.
3. **Failure taxonomy** — every failure is classified into a stable label set (e.g. `semantic-misread/boundary-rule`) so failures can be compared across runs, models, and prompt revisions.
4. **Regression detection** — N trials per task per configuration, compared as pass rates and drift statistics; never a single-trace verdict.

## Design points

- **Durable records from the first second:** a run writes its record at start (with pid and heartbeat), streams its wire log, and classifies its own ending — completed, interrupted, abandoned, endpoint-timeout. A run cannot hang invisibly, and an interrupted run stays in the denominator as what it is.
- **Oracle controls:** a scripted-correct agent must PASS and a null agent must FAIL before any live result is trusted.
- **Series integrity:** comparison series are keyed on battery + resolved configuration, recomputed on read; the seed is recorded with every run but excluded from series identity. Infrastructure failures are recorded and excluded from grading; model-attributable failures are counted.
- **Sandboxing with honest enforcement:** runs execute under bubblewrap at a fixed workspace, with the enforcement level recorded as data — and where the sandbox cannot be created, tests skip with the reason printed rather than pass vacuously.
- **Importers, not hand-writing:** session logs (agent CLI sessions, raw API message logs) are converted into the trace format by importer; nobody hand-writes traces outside examples and smoke tests.

## What it has measured so far

The first live results are deliberately unflattering — that is what the harness is for. In brief: the pinned reference local model **fails the first graded task** (it computed the correct value and still produced a failing trace); identical-seed determinism runs **diverged**, and the divergence is documented rather than smoothed over; and a trial lost to a machine reboot mid-run is preserved as an *abandoned* record. Detail and caveats: [docs/measured-results.md](docs/measured-results.md).

## Roadmap

Model Watch is next: drift detection (the same battery re-run over time, grade sets compared) and context decay (the same tasks at graded context loads).

## License

See [LICENSE](LICENSE). Published for review and portfolio purposes; all rights reserved.
