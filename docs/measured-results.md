# Measured Results — First Live Runs

What the harness has actually established so far, stated with its limits. These are summarized from the private working repository's build and run records; the underlying traces and battery are not published.

## The instrument, before any subject

- **Suite:** 181 tests, CI green (179 passed, 2 skipped — the skips are sandbox-probe skips on runners where the sandbox cannot be created, with the reason printed).
- **Oracle controls:** a scripted-correct agent passes and a null agent fails, in both directions, before live results were accepted.
- **Durable records:** every run writes its record at start with pid + heartbeat and classifies its own ending. This design exists because an early live run hung silently for over fifteen minutes with nothing on disk; that failure mode is now impossible to miss.

## Result 1 — the reference model fails Task 001

The pinned reference subject (a small local model, `qwen3:8b`) was graded **FAIL** on the battery's first task, with failure labels **missing-call** and **semantic-misread/boundary-rule**. The notable detail: the model stated the correct computed value and *still* failed — its trace wrapped the answer in phrasing that violated the task's boundary rule, and a required tool call never happened. An answer-only grader would have passed it. That gap is the reason this harness grades tool calls rather than final text.

## Result 2 — determinism is empirical, not assumed

Two runs at temperature 0 with the same seed (42) **diverged**. The divergence is documented in the working record, and it changed the design: series are keyed on battery + resolved configuration recomputed on read, the seed is stored with every run but excluded from series identity, and verdicts are multi-trial statistics — never one run, however it was seeded.

## Result 3 — an abandoned run, kept

During the first live battery, the host machine rebooted mid-trial. The trial's record — present, with its heartbeat frozen and its process gone — is preserved as an **abandoned** record and counted as exactly that: not a pass, not a model failure, and not deleted. Denominators in this project include the runs that went wrong.

## What these results do NOT establish

- Nothing about any model beyond the subjects named, on the tasks run, under the configurations recorded. One task is a smoke signal, not a benchmark.
- No rankings. The battery is small and private by design; these pages report the instrument's first findings, not a leaderboard.
