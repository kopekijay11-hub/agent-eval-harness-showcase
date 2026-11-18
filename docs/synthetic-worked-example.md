# SYNTHETIC worked example (invented trace)

**SYNTHETIC / INVENTED TRACE — for illustration only.** This worked example is entirely fabricated for documentation purposes: the scenario, identifiers, timestamps, paths, arguments, outputs, hashes, and results do not come from any real recorded run or live battery.

## Scenario (fictional)

A hypothetical agent is asked to report the number of lines in a workspace file named `notes.txt`. The expected behavior (by type) is to read the file via a tool call and then report the count. The trace below shows one invented, passing-style sequence of events.

## Synthetic trace (JSON)

```json
{
  "header": {
    "schema_version": "0.1.0",
    "signature_algorithm_version": "jcs-sha256-v1",
    "ignore_list_name": "synthetic.example",
    "ignore_list_version": "1.0.0",
    "run_id": "run-synthetic-0001",
    "trial_id": "trial-synthetic-0001",
    "batch_id": "batch-synthetic-demo",
    "agent_id": "synthetic-demo-agent",
    "model_id": "synthetic-model",
    "model_version": "0.0-synthetic",
    "prompt_hash": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "prompt_hash_algorithm": "sha256",
    "tools": [
      {
        "name": "read_file",
        "description": "Read a file from the workspace.",
        "parameters": {
          "type": "object",
          "properties": {
            "path": { "type": "string" }
          },
          "required": ["path"]
        }
      }
    ],
    "redaction": "none"
  },
  "events": [
    {
      "seq": 1,
      "type": "tool_call",
      "timestamp": "2026-01-01T00:00:01Z",
      "tool_call_id": "tc-synth-1",
      "name": "read_file",
      "arguments": {
        "path": "/workspace/notes.txt"
      },
      "call_signature": "read_file:bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
    },
    {
      "seq": 2,
      "type": "tool_result",
      "timestamp": "2026-01-01T00:00:01Z",
      "tool_call_id": "tc-synth-1",
      "status": "ok",
      "is_error": false,
      "truncated": false,
      "duration_ms": 2,
      "output": "alpha\nbeta\ngamma\n"
    },
    {
      "seq": 3,
      "type": "message",
      "timestamp": "2026-01-01T00:00:02Z",
      "content": "The file notes.txt contains 3 lines."
    }
  ]
}
```

## What this illustrates (tool-call grading view)

- **Tool identity & order:** a `read_file` tool call (seq 1) precedes the concluding message (seq 3), matching a read-then-report pattern.
- **Arguments/signature:** the call records raw `arguments` and a `call_signature` (shown here as a placeholder-form hash in this invented example) as the equality key concept.
- **Linking:** the `tool_result` (seq 2) references `tool_call_id` `tc-synth-1`.
- **Grading note:** in the real harness, pass/fail is determined against task criteria (including call signatures/order) and reported as pass/fail outcomes; this synthetic example is illustrative only and not a scored result.
