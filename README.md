# Crinkl

**Turn the receipt shoebox into reconciled books.**

Crinkl is a free, open-source tool for small businesses. You photograph your paper receipts and upload your bank statements. Crinkl reads both, matches each bank charge to its receipt, and gives you a clear list of what matched, what's missing a receipt, and what needs a human look.

> Status: **pre-alpha** — Phase 0 (foundations). Nothing runs yet.

## How it works (planned v0.1)

1. **Upload a statement** — CSV or PDF from online banking.
2. **Upload receipt photos** — one receipt per photo.
3. **Extraction** — images are cleaned up, OCR'd, and parsed by rules. An LLM is used only when OCR read the text but the rules couldn't structure it, and it only ever sees redacted text.
4. **Matching** — rule-based scoring on amount, date window, vendor name, and card last-4.
5. **Review** — anything low-confidence lands in a review queue.
6. **Export** — CSV of matched / missing receipt / ambiguous / unmatched (possible cash).

## Why it exists

Plenty of small businesses still run on paper receipts. Someone at tax time sits with a bank statement and a pile of paper and matches them by hand. Crinkl does the matching and leaves the human the exceptions.

## Design principles

- **Rules first, LLM where rules run out.** Cheaper, testable, explainable.
- **Measured, not guessed.** Every extraction and matching change is scored against a labeled eval set in CI.
- **Financial data is treated like financial data.** Tenant isolation enforced by the database, app-level encryption, minimal data sent to third parties.
- **Free to run.** Built entirely on free tiers.

Architecture decisions live in [`docs/decisions/`](docs/decisions/). The phased plan is in [`docs/roadmap.md`](docs/roadmap.md).

## Repo layout

```
src/crinkl/   core library: statements, receipts, matching (no web code)
api/          FastAPI backend + job worker
web/          React frontend
evals/        labeled datasets + eval harness
docs/         roadmap and architecture decision records
```

## License

TBD (decision pending).
