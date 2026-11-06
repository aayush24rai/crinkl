# CLAUDE.md — working agreement for AI coding agents

Read this file and everything in `docs/decisions/` before making changes.

## What Crinkl is

A free, multi-tenant web app for small businesses that matches paper receipts (photographed) to bank statement transactions, flags missing receipts, and routes low-confidence results to a human review queue. See `README.md` and `docs/roadmap.md`.

## Who decides what

- **Aayush is the product owner and architect.** Architecture, data model, security model, and scope decisions are his.
- **Do not make architecture decisions silently.** If a task needs one that isn't covered by an accepted record in `docs/decisions/`, stop and propose options with tradeoffs instead of picking.
- **Do not add dependencies, services, or paid tools** without asking. Everything must run on free tiers.
- Requirements come from the product owner, not from any single user's workflow. Do not add features shaped around one business.

## Architecture boundaries

- `src/crinkl/` is a **pure library**: statement parsing, receipt extraction, matching. No FastAPI, no database, no network calls except through an injected LLM client interface. It must be runnable and testable from the command line.
- `api/` wraps the library: HTTP endpoints, auth, Postgres, job queue, storage, encryption.
- `web/` is the React frontend. It talks only to `api/`.
- `evals/` scores the library against labeled ground truth. It never uses real user data.

## Non-negotiable rules

1. **Tenant isolation:** every tenant-owned table has `business_id` and a Postgres Row-Level Security policy (ADR 0002). Never query with a role that bypasses RLS from request-handling code.
2. **No raw images or unredacted text to third parties** (ADR 0004). The only thing that may leave the system is redacted OCR text, and only for businesses that opted in.
3. **Never store full card numbers.** Last 4 digits only.
4. **Sensitive fields and files are encrypted at the application level** with per-business data keys (ADR 0005).
5. **Never use real user data in tests, fixtures, logs, or evals.** Logs must not contain receipt text, amounts tied to vendors, or file contents.
6. **Any change to extraction or matching must keep the eval suite passing** and should report its metric delta in the PR description.

## Conventions

- Python 3.12, type hints everywhere, `ruff` for lint + format, `pytest` for tests (tooling arrives in Phase 0.2).
- Small PRs, one concern each. PR description: what, why, how tested, eval delta if relevant.
- New architecture decision → new file in `docs/decisions/` using the template in `docs/decisions/README.md`.

## Current phase

Phase 0 — Foundations. See `docs/roadmap.md`.
