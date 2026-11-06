# Roadmap

Each sub-phase has a "done when." Move on only when it's met.

## v0.1 — MVP

### Phase 0: Foundations
- **0.1** Repo, layout, `CLAUDE.md`, decision records for everything decided so far. *Done when:* a fresh agent session can read the repo and explain the architecture back.
- **0.2** Tooling: Python env, ruff, pytest, GitHub Actions running tests on each PR. *Done when:* CI is green on an empty test.

### Phase 1: Eval data first
- **1.1** Collect ~50–100 receipts (own + synthetic; clean, crumpled, faded, long) and 2–3 statements (CSV + PDF, synthetic or redacted).
- **1.2** Label ground truth: vendor, date, total, tax, last-4 per receipt; correct receipt↔transaction matches.
- **1.3** Eval harness skeleton. *Done when:* it runs in CI and prints metrics, even against a stub extractor.

### Phase 2: Statement ingestion (library)
- **2.1** CSV parser → normalized transaction schema.
- **2.2** PDF parser for 1–2 bank layouts.
- **2.3** Eval: transaction-level parse accuracy. *Done when:* every test statement parses correctly.

### Phase 3: Receipt extraction (library)
- **3.1** Image preprocessing (deskew, crop, contrast, binarize).
- **3.2** OCR + OCR-quality scoring.
- **3.3** Rule-based field parsing with per-field confidence.
- **3.4** Redaction of OCR text (card numbers, addresses, names).
- **3.5** LLM fallback for good-OCR / bad-parse, structured JSON, rate-limited.
- **3.6** Failure routing: retake / LLM / review queue.
- **3.7** Eval: field-level accuracy rules-only vs rules+LLM, failure-type breakdown. *Done when:* real numbers exist and failure modes are known.

### Phase 4: Matching engine (library)
- **4.1** Scoring: amount (tip tolerance), date window, fuzzy vendor, last-4.
- **4.2** Classification: matched / ambiguous / missing receipt / unmatched receipt.
- **4.3** Eval: match precision/recall. *Done when:* CLI run (statement + receipts → report) works end to end.

### Phase 5: Backend platform
- **5.1** FastAPI skeleton, Postgres schema, migrations.
- **5.2** Auth, one business per account.
- **5.3** RLS policies + a test proving Business A can't read Business B.
- **5.4** Postgres job queue + worker, status, retries.
- **5.5** R2 storage + envelope encryption.
- **5.6** Endpoints: upload, job status, results, review actions, export.
- **5.7** Structured logging, error tracking, audit log.
- **5.8** Per-business quotas (processing + storage), account and data deletion.
- **5.9** Archive API (ADR 0007): browse/filter, single-file download, yearly "audit pack" export (zip of receipts in folders + CSV index), per-file and per-year deletion. *Done when:* a year of test receipts exports, re-imports cleanly into a spreadsheet, and every file opens; search on a synthetic 10,000-receipt business stays within the latency budget.

### Phase 6: Frontend
- **6.1** Auth screens.
- **6.2** Upload flow with progress and retake prompts.
- **6.3** Review queue: receipt image beside transaction; approve / fix / reject.
- **6.4** Results view + CSV export.
- **6.5** Archive view: browse by year / month / vendor / status, open a receipt beside its transaction, download, export audit pack.

### Phase 7: Deploy and operate
- **7.1** Deploy backend, frontend, DB on free tiers.
- **7.2** Secrets and environment config.
- **7.3** Monitoring: health check, error alerts, job failure rate.
- **7.4** Privacy page, terms, honest security model in README.

### Phase 8: Launch v0.1
- **8.1** First small businesses try it.
- **8.2** Feedback, published eval results, demo video.

## v0.2
- LLM agent with tools that resolves **ambiguous matches only**, plus an eval showing whether it beats rules alone.
- More bank layouts.

## v0.3
- Organizations with multiple businesses (only if real users ask).
- Optional sync of the archive to the user's own Google Drive (candidate, ADR 0007).
- Multiple receipts per photo.
- Line items.

## v1.0
- QuickBooks export, email ingestion, categorization.
