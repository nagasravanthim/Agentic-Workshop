---
id: SPEC-epic-1
companions: [triage-decision-schema.md, ../../../mcp/triage_server.py]
sources: [../../../INTENT.md]
---

> **Canonical contract.** This SPEC and the files in `companions:` are the complete, preservation-validated contract for what to build, test, and validate. Source documents listed in frontmatter are for traceability — consult them only if you need narrative rationale or prose color this contract intentionally omits.

# Epic 1: triage data and schema

## Why

Later epics need a shared triage-decision shape and a local copy of workshop ticket data. Epic 1 establishes both: a strict JSON schema every downstream consumer can validate against, and an idempotent loader that turns read-only seed CSVs into `app.db` for the existing MCP triage server—without touching the agent, evals, or network.

## Capabilities

- **CAP-1**
  - **intent:** Callers can produce and validate triage decisions that always use the same four fields and allowed values.
  - **success:** A decision such as `billing` / `P2` / `billing-team` with a one-sentence rationale passes validation. Values outside the allowed sets, missing fields, extra fields, or a non–one-sentence rationale fail with a clear error. Field rules and enums are defined in `triage-decision-schema.md`.

- **CAP-2**
  - **intent:** A person can materialize workshop seed data into SQLite with one command, repeatedly, for local development and MCP lookups.
  - **success:** `uv run python load_seed.py` creates `app.db` with `tickets` and `customers` tables whose columns match `seed/tickets.csv` and `seed/customers.csv`. A second run produces the same database as the first (no duplicate rows or drift). `mcp/triage_server.py` can query `ticket_id`, `customer_id`, `created_at`, `text` on `tickets` and `customer_id`, `name`, `plan`, `open_tickets` on `customers`.

## Constraints

- Python 3.12 or newer, managed with uv; add packages with `uv add`, not pip.
- `seed/` is read-only; the loader reads only `seed/tickets.csv` and `seed/customers.csv`.
- No network calls and no API keys in this epic.
- `mcp/triage_server.py` is read-only; table and column names it already uses must keep working.

## Non-goals

- The LangChain triage agent, changes to MCP tools, the Epic 3 eval harness, and any user interface.

## Success signal

`uv run pytest` (or equivalent project tests) shows the triage schema accepting valid decisions and rejecting invalid ones, and `uv run python load_seed.py` run twice leaves `app.db` unchanged—so Epic 2 can rely on the schema and MCP reads without re-seeding logic.

## Assumptions

- Category and route are validated independently against their enums; INTENT lists matching pairs for human clarity, but validation does not auto-derive route from category unless tests explicitly require that stricter rule later.
