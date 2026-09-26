# Triage decision schema

Every triage decision is one JSON object with exactly these fields:

| Field | Type | Rules |
|-------|------|--------|
| `category` | string | One of: `billing`, `bug`, `access`, `performance`, `how-to` |
| `priority` | string | One of: `P1`, `P2`, `P3`, `P4` |
| `route` | string | One of: `billing-team`, `bug-team`, `access-team`, `performance-team`, `how-to-team` |
| `rationale` | string | Exactly one sentence (a single line of prose ending with terminal punctuation) |

Validation rejects any object that is missing a field, adds extra fields, uses values outside the allowed sets, or supplies a rationale that is not a single sentence. Rejection must surface a clear error (message or exception) that identifies what failed.

Expected pairings (for tests and examples; validation is per-field enum membership, not automatic cross-field routing):

| category | route |
|----------|-------|
| billing | billing-team |
| bug | bug-team |
| access | access-team |
| performance | performance-team |
| how-to | how-to-team |
