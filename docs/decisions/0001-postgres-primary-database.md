# 0001. Postgres as the primary database

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
The core data is relational: business → statements → transactions ↔ matches ↔ receipts. We need joins, multi-row transactions (a match and both status updates succeed or fail together), constraints, and database-enforced tenant isolation. Must run on a free tier.

## Options considered
1. **Postgres** — relational, transactional, Row-Level Security, `JSONB` for raw extraction output, can also host the job queue. Free tier on Neon.
2. **MongoDB (document store)** — flexible schema, but the data is highly relational, no direct RLS equivalent, cross-document consistency is awkward.
3. **Elasticsearch/OpenSearch** — a search engine, not a source of truth. Useful later as a secondary index fed from Postgres.

## Decision
Postgres (Neon free tier) as the single source of truth.

## Consequences
- One database for data, tenancy enforcement, and the job queue.
- Schema migrations required from day one.
- Revisit search if receipt search becomes a real need and Postgres full-text search can't keep up — add a search index then, fed from Postgres.
