# Salon Assistant — Luna

A Python salon booking assistant built for the Week 16 assignment.

Luna helps customers browse services and available slots, book appointments,
reschedule bookings, and cancel appointments. The model only **proposes**;
Python code validates salon rules and explicit customer confirmation before
any booking change is executed.

## Main file

- `HW_Week16_SalonAssistant.ipynb` — implementation, tests, live-model
  experiments, and report.

## Status

- [x] `cancellation_fee_cents` — cancellation fee based on time remaining
- [x] `interpret_reply` — conservative yes/no/unclear classifier
- [x] `TOOL_SCHEMAS` — seven allowed tools in OpenAI format
- [x] Helper functions: `_redact`, `_euro`, `_validate_args`, `_result`
- [x] `SalonAssistant` — main controller class
- [ ] 22 public scenarios passing
- [ ] Live-model experiments (Section C)
- [ ] Personal test scenarios (Section B)
- [ ] Final report (Section D)

## Project layout

| File | Purpose |
|---|---|
| `HW_Week16_SalonAssistant.ipynb` | Main notebook: implementation, tests, report |
| `given.ipynb` | Provided salon backend and test harness |
| `requirements.txt` | Python dependencies |
| `.gitignore` | Ignore rules |


## Architecture

`SalonAssistant` is the controller between the LLM and the salon backend.
The model only **proposes**; this class validates rules, asks for customer
confirmation, executes changes with idempotency, and never lets the model
claim something the system did not confirm.

### Session state

Each session stores `customer` (from the app, never the model), `messages`
(history without the system prompt), `pending` (one proposal awaiting
confirmation), `turn`, `seq`, and `seen_ids`.

### Key methods

| Method | What it does |
|---|---|
| `handle` | Adds the user message, reads and clears `pending`, executes it if the customer said yes, then calls the model loop. |
| `_system_prompt` | Builds the system prompt with `Now: YYYY-MM-DD HH:MM (Weekday)` from the injected clock. |
| `_build_messages` | Trims history to ≤ 24 messages, keeping whole turns so no `tool` is orphaned. |
| `_run_model` | Loops up to 6 model calls: executes tool calls or returns the final reply; falls back safely otherwise. |
| `_valid_batch` | Rejects the whole tool batch if any entry is malformed, over the limit, or reuses an id. |
| `_run_tool_calls` | Emits one assistant message with all tool calls, then one `tool` message per result. |
| `_record` / `_audit` | Write the tool observation and a redacted audit event (8 keys). |
| `_run_one` | Validates tool name, JSON, dict shape, and schema before dispatch. |
| `_t_list_services` / `_t_find_slots` / `_t_get_my_bookings` | Read tools; identity comes from the session. |
| `_t_book` | Six pre-checks before creating a `pending` proposal — never books directly. |
| `_alternatives` | Ranks free slots by distance from the requested time, returns up to 3. |
| `_execute_pending` | Runs the confirmed action with a deterministic `request_key`, then records the result. |

### Rules implemented

- **R1 (identity):** `st['customer']` from `open_session`.
- **R2 (confirmation):** `handle` + `pending` + `interpret_reply`.
- **R3 (execution protocol):** synthetic assistant + tool messages in `_execute_pending`.
- **R4 (pre-check):** six checks in `_t_book` before any proposal.
- **R5 (idempotency):** `request_key` from `(session, turn, seq)`.
- **R6 (errors):** partial — retry and `find_by_key` still to come.
- **R7 (limits):** `_valid_batch` + `_run_model`.
- **R8 (grounding):** `_check_grounding` still to come.
- **R9 (history):** `_build_messages` + `_system_prompt`.
- **R10 (audit):** `_audit` with 8 keys, `_redact`, 80-char preview.
- **R12 (honesty):** `SAFE_FALLBACK` in `_run_model`.