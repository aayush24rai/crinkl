# 0002. Tenant isolation with Postgres Row-Level Security

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
Multiple businesses share one database. A single missed `WHERE business_id = …` would leak one business's financial data to another. Code is partly written by AI agents, which raises the odds of exactly that mistake.

## Options considered
1. **App-level filtering only** — simple, but nothing catches a forgotten filter.
2. **Postgres RLS** — the database filters every query by the business set for the current request, so a buggy query still can't return another tenant's rows. More setup; harder to debug empty results.
3. **Schema per tenant** — strong isolation, painful migrations. Overkill.

## Decision
RLS on every tenant-owned table, keyed on `business_id`. The API sets the current business once per transaction (`SET LOCAL app.business_id = …`) after authentication. Request-handling code connects as a restricted role that cannot bypass RLS; the owner/migration role is used only for migrations.

## Consequences
- Isolation is enforced by the database, not by remembering a filter.
- Required: an automated test proving Business A cannot read or write Business B's rows, run in CI.
- Gotchas to handle: setting the value inside each transaction with connection pooling, and never using the owner role at runtime.
