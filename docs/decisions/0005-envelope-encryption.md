# 0005. App-level envelope encryption with per-business keys

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
We store other people's receipts and bank statements. Provider disk encryption protects against stolen disks, not against a leaked database dump or someone browsing the database. True end-to-end encryption isn't possible because the server must process plaintext (OCR, matching).

## Options considered
1. **Provider encryption at rest only** — free and automatic; anyone with DB access sees everything.
2. **Envelope encryption** — each business has a data key; files and sensitive fields are encrypted with it; data keys are encrypted by a master key held only in the server's secret config. Protects DB dumps and backups.
3. **Keys derived from the user's password** — operator can't read data at rest, but forgotten password = data lost, and background jobs can't run while the user is offline.
4. **All processing in the browser** — near-zero operator access, but weak OCR and impractical for the LLM step.

## Decision
Option 2 on top of provider encryption:
- Encrypt receipt images, statement files, and sensitive fields (vendor, amounts tied to vendors, OCR text).
- Master key lives in hosting secret environment variables (free); no paid KMS for now.
- Data minimization: store card last-4 only; offer auto-deletion of raw images after processing.
- Audit log of data access; operator rule: never query production data for debugging.

## Consequences
- Encrypted fields can't be filtered in SQL — matching runs in application code after decrypting one business's data.
- If the master key leaks, all data keys are exposed. Upgrade path: a managed KMS.
- The README must state plainly that the operator can technically decrypt, and how that access is restricted. No "zero-access" claims.
