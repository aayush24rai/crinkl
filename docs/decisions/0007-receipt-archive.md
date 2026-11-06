# 0007. Receipt and statement archive

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
Businesses need somewhere to keep every scanned receipt and statement, not just the reconciliation result. They need to find a receipt months later (audits, client questions) and keep records for years. Tax record-keeping periods are commonly several years; Crinkl should not decide retention for users.

Constraints:
- Free tier: Cloudflare R2 gives 10 GB of storage per month. A compressed receipt photo is roughly 150–300 KB, so 300 receipts ≈ 45–90 MB. That's on the order of 100+ business-years of receipts before hitting the limit.
- ADR 0005 encrypts files and sensitive fields, so the database can't filter or search encrypted fields directly.
- Crinkl is a free, single-maintainer project. If it ever shuts down, users must not lose their records.

## Proposed scope (v0.1)
- Every uploaded receipt and statement is kept, encrypted, in R2, linked to its extracted data and match.
- Browse by year / month / vendor / match status; open a receipt next to its bank transaction.
- Download any single file.
- **Export ("audit pack"):** one zip per year with receipts in `year/month/` folders (named like `2026-03-14_HomeDepot_84.12.jpg`) plus a CSV index of every receipt and its matched transaction.
- Retention is the user's choice: keep by default, delete per file or per year, account deletion removes everything. Auto-deleting images after processing stays an opt-in only.
- Per-business storage quota.

## Open questions

### Q1. What gets stored per receipt? — **Decided: both**
1. **Original photo only** — the true record; largest files.
2. **Cleaned image only** (cropped, deskewed, compressed) — smaller and easier to read; the original isn't kept, which may matter in an audit.
3. **Both** — safest; about double the storage.

**Decision:** store the original, uncropped photo untouched as the record of truth, plus a cleaned copy for viewing and OCR. Both encrypted. Roughly doubles storage (still ~50+ business-years within the free tier). The original is never modified or deleted by processing.

### Q2. How does search work with encrypted fields? — **Decided: decrypt-then-filter**
1. **Decrypt-then-filter in app code** — simple; fine for one business's few thousand receipts; gets slow at large volume.
2. **Leave date and amount unencrypted, encrypt vendor and OCR text** — date/amount filters run in SQL; dates and amounts alone reveal little without the vendor, but it is more exposure than ADR 0005 currently allows.
3. **Blind index** (store a keyed hash of the normalized vendor name for exact-match lookups) — searchable without plaintext; no fuzzy search; more complexity.

**Decision:** option 1. Search loads one business's records (already scoped by RLS), decrypts them, and filters, sorts, and paginates in application code. Fuzzy vendor search comes for free. ADR 0005's encryption scope is unchanged.

**Revisit trigger:** a performance test searches a synthetic business with 10,000 receipts and fails if it exceeds a set latency budget (to be set in Phase 5.9). If that test or real usage fails, move to the hybrid: unencrypted date/amount for SQL narrowing, then decrypt-and-fuzzy-match vendor on the small result set. That change would supersede part of ADR 0005 with a new record.

### Q3. How do we protect users against Crinkl going away? — **Decided: export + reminder**
1. **Export only** — users can always download everything; we prompt a yearly export.
2. **Export + scheduled reminder** to download the audit pack (e.g. after each tax year).
3. **Export + optional sync** to the user's own Google Drive / Dropbox — the strongest guarantee; adds OAuth and an integration to maintain (v0.3+ candidate).

**Decision:** option 2 for v0.1 — full export always available, plus a reminder to download the audit pack. Optional Google Drive sync is a candidate for a later version, not committed.

## Consequences
- Crinkl becomes a system of record, not just a matcher. Durability and export matter as much as accuracy.
- Privacy page must state where files live, how long they're kept, and how to get them out.
- Storage quota becomes a real limit to communicate in the UI.
