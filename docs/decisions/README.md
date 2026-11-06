# Architecture Decision Records

One file per decision. Never edit an accepted record's decision — supersede it with a new record and link both ways.

| # | Decision | Status |
|---|---|---|
| [0001](0001-postgres-primary-database.md) | Postgres as the primary database | Accepted |
| [0002](0002-row-level-security.md) | Tenant isolation with Postgres Row-Level Security | Accepted |
| [0003](0003-postgres-job-queue.md) | Postgres-backed job queue, no Redis | Accepted |
| [0004](0004-llm-data-policy.md) | LLM sees redacted OCR text only, routed by failure type | Accepted |
| [0005](0005-envelope-encryption.md) | App-level envelope encryption with per-business keys | Accepted |
| [0006](0006-single-business-tenancy.md) | One business per account for v0.1 | Accepted |

## Template

```markdown
# NNNN. Title

- Status: Proposed | Accepted | Superseded by NNNN
- Date: YYYY-MM-DD
- Decider: Aayush Rai

## Context
What problem, what constraints.

## Options considered
1. Option — pros / cons
2. Option — pros / cons

## Decision
What we chose.

## Consequences
What gets easier, what gets harder, when we'd revisit.
```
