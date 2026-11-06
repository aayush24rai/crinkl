# 0006. One business per account for v0.1

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
Some users are a single business; others (bookkeepers, small accounting firms) manage many businesses. Supporting both from day one adds an organization layer and an extra permission check.

## Options considered
1. **One account = one business** — simplest data model and permissions.
2. **Organization → many businesses** — supports bookkeepers; one more table and one more check on every request.

## Decision
One business per account for v0.1. Keep `business_id` as the tenancy key everywhere so an organization layer can be added above it later without changing RLS policies.

## Consequences
- Bookkeepers serving many clients need multiple accounts in v0.1.
- Revisit in v0.3 only if real users ask for it.
