# Trace format

One recorded agent run = one trace file. The machine-readable contract is
`schema/trace.schema.json` (JSON Schema draft 2020-12); this document explains
each field in plain language.

## Top-level layout

A trace has exactly two top-level sections: `header` and `events`.

## Header (run metadata)

- `schema_version` — version of this trace format (string).
- `signature_algorithm_version` — version of the call_signature algorithm
  (e.g. `jcs-sha256-v1`), so old and new traces can be distinguished.
- `run_id` — unique id for this single run.
- `trial_id` — id of the trial within a batch (traces group by batch and trial
  per the multi-trial regression design).
- `batch_id` — id of the batch this run belongs to.
- `agent_id` — id of the agent under test.
- `model_id` — id of the model backing the agent.
- `model_version` — version of the model (kept separate from model_id).
- `temperature` — sampling temperature, when the agent exposes it (optional).
- `seed` — random seed, when the agent exposes it (optional).
- `prompt_hash` — sha256 hex digest of the prompt text. Prompts are referenced
  by content hash, never by name alone.
- `prompt_hash_algorithm` — algorithm used for prompt_hash (`sha256`).
- `tools` — snapshot of the tool definitions the agent was given for this run:
  each entry has a name and an argument schema (JSON Schema fragment).
- `redaction` — redaction marker: `none`, `partial`, or `full`, recording whether
  the trace was scrubbed for sharing.
- `ignore_list_name` / `ignore_list_version` — name and version of the
  argument ignore-list actually applied when computing this run's
  call_signatures, so the blanking rule can be audited years later.
- `redacted_fields` — list of the field names / categories removed by
  redaction (categories such as home-directory, secret, token), in the
  order removed. Present only when redaction touched the trace.
- `source_format` / `source_file_sha256` / `importer_version` — provenance
  for imported traces: the format the trace was imported from, the SHA-256
  of the source file (computed before and after the import run; they must
  match, proving the source was read-only), and the importer version.

## Events

`events` is a time-ordered array. Every event carries:

- `seq` — monotonic integer sequence number. This is the order axis; it is
  separate from `timestamp`.
- `timestamp` — ISO-8601 wall-clock time of the event.
- `type` — event kind (e.g. `tool_call`, `tool_result`, `message`).

Tool calls follow the MCP tool-call shape: a `name` plus an `arguments` object.

### tool_call event

- `tool_call_id` — unique id for this call. The matching `tool_result` carries
  the same id, so parallel calls are linkable.
- `name` — tool name (MCP shape).
- `arguments` — the RAW arguments object passed by the agent, stored verbatim.
- `call_signature` — `name` + SHA-256 hash of the arguments canonicalized per
  RFC 8785 (JCS), stored as `name:<sha256-hex>`. The raw arguments are stored
  alongside the hash; the hash is a compact equality key, not a replacement.
  The schema pins the pattern `^.+:[0-9a-f]{64}$`. Because MCP tool names may
  be namespaced and contain colons, a signature is always parsed by splitting
  on the LAST colon -- a SHA-256 hex digest never contains one.

Canonicalization ignore-list: before hashing, the following volatile argument
fields must be blanked so that environment-specific values do not change the
signature: temporary file paths, timestamps, and generated ids. Blanked fields
are replaced with `null`. The exact field names are fixed per tool in the
importer/tool registry (future work) and recorded in the header as
`ignore_list_name` + `ignore_list_version`.

The arguments are canonicalized with a real RFC 8785 (JCS) implementation
(pinned dependency `rfc8785`, proven against the RFC vector suite under
tests/fixtures/jcs-vectors/). The signature function fails loudly -- it raises
an explicit error instead of hashing quietly -- on values JCS cannot represent:
`NaN`, `Infinity`, and integers outside the ECMAScript safe-integer range
(|v| > 2**53 - 1).

### Why JCS byte-identity matters

JCS is mandated here, not offered as a pluggable option, because call
signatures must be byte-identical across implementations:

* Python and ECMAScript format numbers differently. Python prints `1e+20`;
  JCS/ECMAScript requires `100000000000000000000`. A plain
  `json.dumps(sort_keys=True)` canonicalization would not match a JavaScript
  implementation.
* OpenCode is TypeScript. A future capture path will hash arguments in
  JavaScript; only byte-identical canonicalization makes Python-computed and
  JS-computed `call_signature` values agree.

### tool_result event

- `tool_call_id` — id of the call this result answers (same id as the
  `tool_call`).
- `status` — result status (string).
- `is_error` — boolean error flag.
- `truncated` — boolean flag: was this result cut down before recording?
- `duration_ms` — call duration in milliseconds (number).
- `output` — result payload (optional; may be omitted or redacted).

## Design rules

1. **Traces are IMMUTABLE once written.** A trace is a record of what happened;
   it is never corrected, annotated, or enriched in place.
2. **Grades live in separate grade files.** A grade file references its trace by
   the trace's content hash (sha256 of the trace file bytes). Grades are never
   embedded inside a trace.

## Future replay matching rule (Phase 2+)

When replaying a trace against a live agent, calls are matched by
`call_signature` plus the occurrence index among identical signatures (the first
`read_file:{hash}` matches the first, the second matches the second, etc.).
Call order is graded separately by the grader, not by the matching key. An
unmatched live call produces an explicit `replay-miss` event and never a
fabricated result.
