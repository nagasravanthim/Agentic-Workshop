---
title: '1 Triage decision schema'
type: 'feature'
created: '2026-09-26'
status: 'done'
route: 'dispatch'
review_loop_iteration: 0
baseline_commit: '4cc901424a83439c887129e02af0f772ffcac204'
context:
  - '{project-root}/_bmad-output/specs/spec-epic-1/SPEC.md'
  - '{project-root}/_bmad-output/specs/spec-epic-1/triage-decision-schema.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** Epic 2 needs a single, strict triage-decision shape every caller can validate. Nothing in the repo defines that schema yet.

**Approach:** Add a small `triage` package with a Pydantic model and a `validate_decision` entry point that accepts JSON (str/bytes) or dict, enforces the enums and one-sentence rationale from `triage-decision-schema.md`, and raises a dedicated error type with messages that name the failing field.

## Boundaries & Constraints

**Always:** Match allowed values and field rules in `triage-decision-schema.md`; use `extra="forbid"` on the model; reject missing/extra fields; no network or API keys; do not edit `seed/`, `mcp/triage_server.py`, or Epic 2+ files.

**Never:** Implement `load_seed.py` (story 2); build the agent; auto-correct category/route mismatches unless explicitly tested.

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|----------|--------------|---------------------------|----------------|
| HAPPY_PATH | Valid dict or JSON string for billing/P2/billing-team + one-sentence rationale | `TriageDecision` instance with same values | N/A |
| BAD_ENUM | Unknown category, priority, or route | No decision returned | `TriageValidationError` naming the field |
| BAD_RATIONALE | Empty, whitespace-only, or multi-sentence rationale | No decision returned | `TriageValidationError` mentioning rationale |
| MISSING_FIELD | Object omitting any required key | No decision returned | `TriageValidationError` naming missing field |
| EXTRA_FIELD | Object with unknown key | No decision returned | `TriageValidationError` mentioning extra/forbidden field |
| NOT_OBJECT | Non-JSON text, non-object JSON, bytes garbage | No decision returned | `TriageValidationError` explaining invalid JSON or non-object |

</frozen-after-approval>

## Code Map

- `pyproject.toml` -- declares `pydantic` and `pytest`; no package layout yet for `triage`.
- `tests/` -- directory missing; create `tests/test_schema.py` (and `tests/conftest.py` only if needed).
- `_bmad-output/specs/spec-epic-1/triage-decision-schema.md` -- authoritative enum and rationale rules for tests.
- `run_agent.py` / `mcp/triage_server.py` -- out of scope; do not modify in this story.

## Tasks & Acceptance

**Execution:**
- [x] `triage/schema.py` -- add `TriageDecision` model, `TriageValidationError`, and `validate_decision()` per matrix -- CAP-1 contract for Epic 2 structured output.
- [x] `triage/__init__.py` -- export public symbols if useful for imports -- keeps import path stable for later epics.
- [x] `tests/test_schema.py` -- parametrize accept/reject cases from the I/O matrix -- proves CAP-1 success criteria.

**Acceptance Criteria:**
- Given a valid decision payload, when `validate_decision` runs, then it returns a frozen `TriageDecision` with matching field values.
- Given each matrix error case, when `validate_decision` runs, then it raises `TriageValidationError` with a message that identifies the problem (field name or JSON issue).
- Given the story is complete, when `uv run pytest tests/test_schema.py` runs, then all tests pass.

## Implementation Notes

- Added `tests/conftest.py` to put repo root on `sys.path` so `triage` imports work under pytest (matches stage-2 workshop layout).
- `uv run pytest tests/test_schema.py` — 16 passed.

## Verification

**Commands:**
- `uv run pytest tests/test_schema.py` -- expected: all tests pass with no failures.

### Review Findings

- [x] [Review][Decision] One-sentence detection vs abbreviations — resolved: end-anchored detection with abbreviation-aware `_SENTENCE_BREAK` plus `_GLUED_SENTENCE` for punctuation without whitespace.

- [x] [Review][Patch] Enforce terminal punctuation on rationale [`triage/schema.py:28-34`]

- [x] [Review][Patch] Reject glued multi-sentence rationales [`triage/schema.py:32-33`]

- [x] [Review][Patch] Assert full model equality for JSON input [`tests/test_schema.py:12`]

- [x] [Review][Patch] Assert identifying messages for NOT_OBJECT cases [`tests/test_schema.py:38-41`]

- [x] [Review][Patch] Cover empty-string rationale in BAD_RATIONALE tests [`tests/test_schema.py:21`]

- [x] [Review][Defer] UTF-8 mojibake in new BMad markdown (`ΓÇö` / `nonΓÇô`) [`_bmad-output/specs/spec-epic-1/SPEC.md`] — deferred: documentation encoding only; no runtime impact.

#### Rejected

- **CAP-2 / `load_seed.py` missing from branch** — `false`: Story 1 scope is CAP-1 only; epic success signal is not this PR's acceptance bar.
- **No test proving mismatched category/route is allowed** — `false`: SPEC assumption explicitly allows independent enum validation.
- **Embedded newline rationale unchecked** — `false`: `"Line one.\nLine two."` is rejected by `_SENTENCE_BREAK` (verified).
- **`UserDict` / non-`dict` mappings** — `false`: I/O matrix specifies dict or JSON only.
- **`pyproject.toml` package layout for `triage`** — `low`: `tests/conftest.py` path bootstrap is documented in the story; not worth packaging churn for this epic.
- **Parametrize happy path across all enum values** — `low`: one valid example plus enum rejection tests suffice for CAP-1.
- **Stale Code Map prose in story body** — `low`: metadata drift only; fix is doc edit, not a code defect.
