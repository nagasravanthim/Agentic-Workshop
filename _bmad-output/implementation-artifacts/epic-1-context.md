# Epic 1 Context: triage data and schema

<!-- Compiled from spec-epic-1. Edit freely. Regenerate if the epic SPEC changes. -->

## Goal

Establish the shared triage-decision JSON shape and validation rules Epic 2+ depend on, plus (in story 2) an idempotent SQLite loader from seed CSVs. This epic does not build the agent, change MCP tools, or use the network.

## Stories

- Story 1: Triage decision schema (CAP-1)
- Story 2: Seed loader (CAP-2)

## Requirements & Constraints

- Decisions use four fields: category, priority, route, one-sentence rationale; invalid input fails with a clear error.
- Allowed values are defined in `triage-decision-schema.md` beside the epic SPEC.
- Python 3.12+, uv for dependencies; `seed/` and `mcp/triage_server.py` stay read-only.
- Story 1 must not require API keys or network access.

## Technical Decisions

- Pydantic is already a project dependency; a small `triage` package at repo root is the natural home for the schema module.
- Category and route are validated as independent enums (no automatic route-from-category enforcement unless tests add it later).

## Cross-Story Dependencies

- Story 2 does not depend on story 1 code, but Epic 2 expects both to exist before the agent ships.
