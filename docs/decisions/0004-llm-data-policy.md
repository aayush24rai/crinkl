# 0004. LLM sees redacted OCR text only, routed by failure type

- Status: Accepted
- Date: 2026-11-05
- Decider: Aayush Rai

## Context
The product should use an LLM for extraction, but the project must run on free tiers, and free LLM API tiers generally allow the provider to use inputs to improve their products, including human review. Receipts contain financial and personal data.

"Low confidence" has two different causes:
- **Good OCR, bad parse** — text was read fine, but rules couldn't structure it (multiple totals, odd layouts, buried vendor names). An LLM on text fixes this.
- **Bad OCR** — blurry, faded, crumpled; the text is garbage. An LLM on that text can't help.

## Options considered
1. **No LLM for real users** — safest, but the product is weaker.
2. **Redacted OCR text to the LLM, never images** — useful for the parse-failure case, limited exposure.
3. **Send images to a vision LLM** — best accuracy on bad photos; sends the most data, can't be reliably redacted.

## Decision
Option 2, with routing by failure type:
- Preprocess every image first (deskew, crop, contrast) to reduce OCR failures.
- **Good OCR, bad parse →** redact (card numbers, addresses, personal names), then send text only to the LLM.
- **Bad OCR →** prompt the user to retake; if still bad, human review queue. No LLM.
- LLM use is a **per-business opt-in**, off by default, with plain-language disclosure of what's sent and to whom.
- Development and evals use only synthetic or the developer's own receipts.

## Consequences
- The eval must report failure-type breakdown (OCR vs parse) and the LLM's lift on parse failures.
- Residual risk: redacted text still goes to a third party on a free tier. Documented honestly in the privacy page.
- Revisit when a paid, no-training API tier is affordable, or if image-level opt-in is requested.
