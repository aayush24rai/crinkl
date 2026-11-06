# 0003. Postgres-backed job queue, no Redis

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
Uploading hundreds of receipts means minutes of OCR and matching — too long for a web request. Work must run in the background with status, retries, and deliberate rate limiting for free-tier LLM quotas. Hosting must be free; a separate always-on Redis service and worker instance would cost money.

## Options considered
1. **Celery + Redis** — industry standard, but needs Redis hosting and a separate worker; sync-first, awkward with async FastAPI.
2. **arq / Dramatiq + Redis** — lighter and async-friendly, still needs Redis.
3. **Postgres-backed queue** (e.g. Procrastinate, or a small custom table using `SELECT … FOR UPDATE SKIP LOCKED`) — no extra service, transactional with the rest of the data, fine at low volume.

## Decision
Postgres-backed queue. Library vs. hand-rolled to be decided in Phase 5.4.

## Consequences
- One fewer service to host and secure.
- Enqueueing a job can share a transaction with the upload record — no "file saved but job lost."
- Doesn't scale to very high throughput. Revisit (move to Redis-based queue) if job volume or latency requirements outgrow Postgres polling.
